---
tags:
  - Java
  - Spring
  - Microservicios
  - Kafka
Creado: 2026-05-21
Relacionado:
  - Microservicios
  - Mensajería
Palabras clave:
  - Kafka, eventos, producer, consumer, topics, particiones, microservicios
---

## 🧠 Por qué Kafka en microservicios

```
Comunicación síncrona (Feign):
  OrderService → UserService → espera respuesta → continúa
  Problema: si UserService está caído, OrderService falla también

Comunicación asíncrona (Kafka):
  OrderService → emite evento OrderCreated → Kafka → UserService lo procesa cuando puede
  OrderService no espera, no le importa si UserService está caído temporalmente
```

## Dependencia

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

## Configuración

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      properties:
        spring.json.add.type.headers: false
    consumer:
      group-id: order-service
      auto-offset-reset: earliest     # earliest (desde el inicio) o latest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "com.empresa.events"
```

## Definir eventos (DTOs de mensajes)

```java
// Convención: en módulo compartido o paquete de eventos
public record OrderCreatedEvent(
    Long orderId,
    Long userId,
    List<OrderItem> items,
    BigDecimal total,
    Instant occurredAt
) {
    public OrderCreatedEvent(Order order) {
        this(order.getId(), order.getUserId(), order.getItems(),
             order.getTotal(), Instant.now());
    }
}

public record PaymentProcessedEvent(
    Long orderId,
    Long paymentId,
    String status,      // SUCCESS, FAILED
    Instant occurredAt
) {}
```

## Producer

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderEventProducer {

    private final KafkaTemplate<String, Object> kafkaTemplate;

    public static final String ORDER_CREATED_TOPIC = "order-created";
    public static final String ORDER_CANCELLED_TOPIC = "order-cancelled";

    public void publishOrderCreated(OrderCreatedEvent event) {
        // key = orderId (para garantizar orden por pedido en la misma partición)
        kafkaTemplate.send(ORDER_CREATED_TOPIC, event.orderId().toString(), event)
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    log.error("Error publicando OrderCreated {}: {}",
                        event.orderId(), ex.getMessage());
                } else {
                    log.info("OrderCreated publicado: {} → partición {}",
                        event.orderId(),
                        result.getRecordMetadata().partition());
                }
            });
    }

    public void publishOrderCancelled(Long orderId, String reason) {
        var event = new OrderCancelledEvent(orderId, reason, Instant.now());
        kafkaTemplate.send(ORDER_CANCELLED_TOPIC, orderId.toString(), event);
    }
}
```

## Consumer

```java
@Service
@Slf4j
public class OrderEventConsumer {

    @KafkaListener(
        topics = "order-created",
        groupId = "payment-service",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void handleOrderCreated(
            OrderCreatedEvent event,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset) {

        log.info("Procesando OrderCreated orderId={} partition={} offset={}",
            event.orderId(), partition, offset);

        try {
            // idempotencia: verificar si ya se procesó
            if (paymentRepository.existsByOrderId(event.orderId())) {
                log.warn("Evento duplicado, ignorando orderId={}", event.orderId());
                return;
            }

            procesarPago(event);

        } catch (Exception ex) {
            log.error("Error procesando OrderCreated: {}", ex.getMessage(), ex);
            throw ex;  // relanzar para que Kafka reintente
        }
    }

    @KafkaListener(topics = "order-cancelled", groupId = "inventory-service")
    public void handleOrderCancelled(OrderCancelledEvent event) {
        inventoryService.liberarStock(event.orderId());
    }
}
```

## Configuración avanzada del Consumer

```java
@Configuration
@EnableKafka
public class KafkaConfig {

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object>
            kafkaListenerContainerFactory(
            ConsumerFactory<String, Object> consumerFactory) {

        var factory = new ConcurrentKafkaListenerContainerFactory<String, Object>();
        factory.setConsumerFactory(consumerFactory);
        factory.setConcurrency(3);  // 3 threads para consumir

        // Error handling
        factory.setCommonErrorHandler(new DefaultErrorHandler(
            new DeadLetterPublishingRecoverer(kafkaTemplate()),
            new FixedBackOff(1000L, 3)  // 3 reintentos con 1s de espera
        ));

        // Batch listener
        factory.setBatchListener(false);

        // ACK manual
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);

        return factory;
    }
}
```

## Dead Letter Topic (DLT)

```java
@KafkaListener(
    topics = "order-created",
    groupId = "payment-service"
)
public void handleOrderCreated(OrderCreatedEvent event) {
    // Si falla 3 veces, va automáticamente a: order-created.DLT
    procesarPago(event);
}

// Consumidor del DLT para manejo manual
@KafkaListener(topics = "order-created.DLT", groupId = "payment-service-dlt")
public void handleDeadLetter(
        OrderCreatedEvent event,
        @Header(KafkaHeaders.DLT_EXCEPTION_MESSAGE) String errorMessage) {
    log.error("Mensaje en DLT: orderId={} error={}", event.orderId(), errorMessage);
    // notificar, guardar en BD de errores, alertar
}
```

## Transaccional Outbox + Kafka

```java
// Garantía: si la BD guarda, el evento SÍ llega a Kafka (eventualmente)
@Transactional
public Order createOrder(CreateOrderRequest req) {
    Order order = orderRepository.save(new Order(req));

    // Guardar en tabla outbox dentro de la misma transacción
    outboxRepository.save(new OutboxEvent(
        "ORDER_CREATED",
        "order-created",
        objectMapper.writeValueAsString(new OrderCreatedEvent(order))
    ));

    return order;
}

// Scheduler que lee outbox y publica en Kafka
@Scheduled(fixedDelay = 1000)
@Transactional
public void processOutbox() {
    outboxRepository.findUnprocessed(PageRequest.of(0, 100))
        .forEach(event -> {
            kafkaTemplate.send(event.getTopic(), event.getPayload());
            event.markProcessed();
            outboxRepository.save(event);
        });
}
```

## Topics — creación programática

```java
@Configuration
public class KafkaTopicConfig {

    @Bean
    public NewTopic orderCreatedTopic() {
        return TopicBuilder.name("order-created")
            .partitions(3)    // más particiones = más paralelismo
            .replicas(1)      // 3 en producción (factor de replicación)
            .config(TopicConfig.RETENTION_MS_CONFIG, "604800000")  // 7 días
            .build();
    }

    @Bean
    public NewTopic orderCancelledTopic() {
        return TopicBuilder.name("order-cancelled")
            .partitions(3)
            .replicas(1)
            .build();
    }
}
```

## 📌 Notas Rápidas
