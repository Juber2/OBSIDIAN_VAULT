---
tags:
  - Java
  - RabbitMQ
  - Spring
  - Mensajería
Creado: 2026-05-21
Relacionado:
  - RabbitMQ Fundamentos
  - Spring Boot
Palabras clave:
  - spring-amqp, RabbitTemplate, @RabbitListener, exchanges, queues, DLQ
---

## Dependencia

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

## Configuración

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual       # ACK manual
        prefetch: 1                    # un mensaje a la vez
        retry:
          enabled: true
          max-attempts: 3
          initial-interval: 1s
          multiplier: 2.0
          max-interval: 10s
```

## Declarar Exchanges, Queues y Bindings

```java
@Configuration
public class RabbitMQConfig {

    // ── Constants ───────────────────────────────────────────
    public static final String ORDER_EXCHANGE = "order.exchange";
    public static final String ORDER_QUEUE = "order.queue";
    public static final String ORDER_DLQ = "order.dlq";
    public static final String ORDER_ROUTING_KEY = "order.created";

    // ── Exchanges ────────────────────────────────────────────
    @Bean
    public TopicExchange orderExchange() {
        return ExchangeBuilder.topicExchange(ORDER_EXCHANGE)
            .durable(true)
            .build();
    }

    @Bean
    public DirectExchange deadLetterExchange() {
        return ExchangeBuilder.directExchange("order.dlx")
            .durable(true)
            .build();
    }

    // ── Queues ────────────────────────────────────────────────
    @Bean
    public Queue orderQueue() {
        return QueueBuilder.durable(ORDER_QUEUE)
            // Dead Letter Exchange: mensajes fallidos van acá
            .withArgument("x-dead-letter-exchange", "order.dlx")
            .withArgument("x-dead-letter-routing-key", "order.dead")
            // TTL del mensaje (opcional)
            .withArgument("x-message-ttl", 3600000)
            .build();
    }

    @Bean
    public Queue deadLetterQueue() {
        return QueueBuilder.durable(ORDER_DLQ).build();
    }

    // ── Bindings ──────────────────────────────────────────────
    @Bean
    public Binding orderBinding(Queue orderQueue, TopicExchange orderExchange) {
        return BindingBuilder
            .bind(orderQueue)
            .to(orderExchange)
            .with(ORDER_ROUTING_KEY);
    }

    @Bean
    public Binding deadLetterBinding(Queue deadLetterQueue, DirectExchange deadLetterExchange) {
        return BindingBuilder
            .bind(deadLetterQueue)
            .to(deadLetterExchange)
            .with("order.dead");
    }

    // ── Serialización JSON ─────────────────────────────────────
    @Bean
    public Jackson2JsonMessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(messageConverter());
        // Publisher Confirms
        template.setConfirmCallback((correlationData, ack, cause) -> {
            if (!ack) {
                log.error("Mensaje no confirmado: {}", cause);
            }
        });
        return template;
    }
}
```

## Producer con RabbitTemplate

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderMessageProducer {

    private final RabbitTemplate rabbitTemplate;

    // Envío básico
    public void publishOrderCreated(OrderCreatedEvent event) {
        rabbitTemplate.convertAndSend(
            RabbitMQConfig.ORDER_EXCHANGE,
            RabbitMQConfig.ORDER_ROUTING_KEY,
            event
        );
        log.info("OrderCreated publicado: {}", event.orderId());
    }

    // Con propiedades del mensaje
    public void publishWithHeaders(OrderCreatedEvent event) {
        MessagePostProcessor postProcessor = message -> {
            message.getMessageProperties().setCorrelationId(UUID.randomUUID().toString());
            message.getMessageProperties().setExpiration("60000");  // TTL 60s
            message.getMessageProperties().setHeader("source", "order-service");
            message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
            return message;
        };

        rabbitTemplate.convertAndSend(
            RabbitMQConfig.ORDER_EXCHANGE,
            RabbitMQConfig.ORDER_ROUTING_KEY,
            event,
            postProcessor
        );
    }

    // Fanout — notificar a todos los suscriptores
    public void publishEvent(String exchange, Object event) {
        rabbitTemplate.convertAndSend(exchange, "", event);  // routing key vacío en fanout
    }
}
```

## Consumer con @RabbitListener

```java
@Service
@Slf4j
public class OrderMessageConsumer {

    // Consumer básico
    @RabbitListener(queues = RabbitMQConfig.ORDER_QUEUE)
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("Procesando order: {}", event.orderId());
        processPayment(event);
    }

    // Con ACK manual
    @RabbitListener(queues = RabbitMQConfig.ORDER_QUEUE)
    public void handleWithAck(
            OrderCreatedEvent event,
            Channel channel,
            @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag) throws IOException {
        try {
            processPayment(event);
            channel.basicAck(deliveryTag, false);  // ACK: procesado OK
        } catch (BusinessException e) {
            // Error de negocio → ACK y no reintentar
            log.warn("Business error: {}", e.getMessage());
            channel.basicAck(deliveryTag, false);
        } catch (Exception e) {
            // Error técnico → NACK para reintentar
            log.error("Error técnico, reencolar: {}", e.getMessage());
            channel.basicNack(deliveryTag, false, true);  // true = requeue
        }
    }

    // Con acceso al mensaje completo
    @RabbitListener(queues = RabbitMQConfig.ORDER_QUEUE)
    public void handleMessage(Message message) {
        String correlationId = message.getMessageProperties().getCorrelationId();
        String source = (String) message.getMessageProperties().getHeader("source");
        log.info("Mensaje de {} correlationId={}", source, correlationId);
    }

    // Declarar queue directamente en el listener (sin @Bean)
    @RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "notification.queue", durable = "true"),
        exchange = @Exchange(value = "order.exchange", type = ExchangeTypes.TOPIC),
        key = "order.*"
    ))
    public void handleNotification(OrderEvent event) {
        notificationService.notify(event);
    }

    // Dead Letter Queue consumer
    @RabbitListener(queues = RabbitMQConfig.ORDER_DLQ)
    public void handleDeadLetter(
            Message message,
            @Header(value = "x-death", required = false) List<Map<String, Object>> xDeath) {
        log.error("Mensaje en DLQ después de {} reintentos",
            xDeath != null ? xDeath.get(0).get("count") : "?");
        // guardar en BD, alertar, analizar
    }
}
```

## RPC Pattern con RabbitTemplate

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final RabbitTemplate rabbitTemplate;

    // Llamada síncrona (espera respuesta)
    public ProductResponse getProduct(Long productId) {
        return (ProductResponse) rabbitTemplate.convertSendAndReceive(
            "product.exchange",
            "product.get",
            productId
        );
    }
}

// En el servicio que responde
@RabbitListener(queues = "product.rpc.queue")
public ProductResponse handleGetProduct(Long productId) {
    return productRepository.findById(productId)
        .map(ProductResponse::from)
        .orElseThrow();
}
```

## Testing con @SpringBootTest

```java
@SpringBootTest
@Testcontainers
class OrderMessagingIntegrationTest {

    @Container
    static RabbitMQContainer rabbitMQ = new RabbitMQContainer("rabbitmq:3.13-management");

    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.rabbitmq.host", rabbitMQ::getHost);
        registry.add("spring.rabbitmq.port", rabbitMQ::getAmqpPort);
    }

    @Autowired
    private OrderMessageProducer producer;

    @Test
    void shouldPublishAndConsume() throws InterruptedException {
        var event = new OrderCreatedEvent(1L, 42L, BigDecimal.TEN);
        var latch = new CountDownLatch(1);

        producer.publishOrderCreated(event);

        assertTrue(latch.await(5, TimeUnit.SECONDS));
    }
}
```

## 📌 Notas Rápidas
