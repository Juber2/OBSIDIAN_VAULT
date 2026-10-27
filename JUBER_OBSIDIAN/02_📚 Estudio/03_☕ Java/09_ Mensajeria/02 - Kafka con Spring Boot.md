---
tags:
  - Java
  - Kafka
  - Spring
  - Mensajería
Creado: 2026-05-21
Relacionado:
  - Kafka Fundamentos
  - Spring Boot
Palabras clave:
  - spring-kafka, KafkaTemplate, @KafkaListener, serialización, error handling
---

## Dependencia

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

## Configuración completa

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092

    # Producer
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      acks: all
      retries: 3
      properties:
        enable.idempotence: true
        max.in.flight.requests.per.connection: 5
        spring.json.add.type.headers: false   # no incluir tipo Java en header

    # Consumer
    consumer:
      group-id: ${spring.application.name}
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      enable-auto-commit: false   # manual commit — más control
      max-poll-records: 500
      properties:
        spring.json.trusted.packages: "com.empresa.events,com.empresa.dto"
        spring.json.value.default.type: com.empresa.events.GenericEvent

    # Listener
    listener:
      ack-mode: MANUAL_IMMEDIATE   # confirmar manualmente
      concurrency: 3               # 3 threads por listener
      missing-topics-fatal: false  # no fallar si el topic no existe al arrancar
```

## KafkaTemplate — producir mensajes

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class EventProducer {

    private final KafkaTemplate<String, Object> kafkaTemplate;

    // Envío simple
    public void send(String topic, Object event) {
        kafkaTemplate.send(topic, event);
    }

    // Envío con key (garantiza orden por key)
    public void send(String topic, String key, Object event) {
        kafkaTemplate.send(topic, key, event)
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    log.error("Error enviando a topic {}: {}", topic, ex.getMessage());
                } else {
                    log.debug("Enviado a topic={} partición={} offset={}",
                        topic,
                        result.getRecordMetadata().partition(),
                        result.getRecordMetadata().offset());
                }
            });
    }

    // Envío a partición específica
    public void sendToPartition(String topic, int partition, String key, Object event) {
        kafkaTemplate.send(topic, partition, key, event);
    }

    // Con headers adicionales
    public void sendWithHeaders(String topic, Object event) {
        ProducerRecord<String, Object> record = new ProducerRecord<>(topic, null, event);
        record.headers()
            .add("source", "order-service".getBytes())
            .add("version", "1.0".getBytes());
        kafkaTemplate.send(record);
    }
}
```

## @KafkaListener — consumir mensajes

```java
@Service
@Slf4j
public class EventConsumer {

    // Consumer básico
    @KafkaListener(topics = "order-created", groupId = "payment-service")
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("Procesando order: {}", event.orderId());
        processPayment(event);
    }

    // Consumer con acceso al ConsumerRecord completo
    @KafkaListener(topics = "order-created")
    public void handleWithRecord(ConsumerRecord<String, OrderCreatedEvent> record) {
        String key = record.key();
        OrderCreatedEvent event = record.value();
        int partition = record.partition();
        long offset = record.offset();
        long timestamp = record.timestamp();

        log.info("key={} partition={} offset={}", key, partition, offset);
        processPayment(event);
    }

    // Consumer con ACK manual
    @KafkaListener(
        topics = "order-created",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void handleWithAck(
            ConsumerRecord<String, OrderCreatedEvent> record,
            Acknowledgment ack) {
        try {
            processPayment(record.value());
            ack.acknowledge();  // confirmar solo si procesó bien
        } catch (BusinessException e) {
            // excepción de negocio → ack igual (no reintentar)
            log.warn("Business error, descartando mensaje: {}", e.getMessage());
            ack.acknowledge();
        }
        // Si hay RuntimeException → no ack → Kafka reintentará
    }

    // Múltiples topics
    @KafkaListener(topics = {"order-created", "order-updated"})
    public void handleOrderEvents(OrderEvent event,
            @Header(KafkaHeaders.RECEIVED_TOPIC) String topic) {
        log.info("Evento del topic: {}", topic);
    }

    // Batch consumer
    @KafkaListener(topics = "order-created",
                   containerFactory = "batchKafkaListenerContainerFactory")
    public void handleBatch(List<OrderCreatedEvent> events) {
        log.info("Procesando batch de {} eventos", events.size());
        processBatch(events);
    }
}
```

## Configuración avanzada del Container Factory

```java
@Configuration
@EnableKafka
public class KafkaConsumerConfig {

    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object>
            kafkaListenerContainerFactory(
            ConsumerFactory<String, Object> consumerFactory) {

        var factory = new ConcurrentKafkaListenerContainerFactory<String, Object>();
        factory.setConsumerFactory(consumerFactory);
        factory.setConcurrency(3);
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);

        // Error handler con Dead Letter Topic
        factory.setCommonErrorHandler(errorHandler());

        return factory;
    }

    @Bean
    public DefaultErrorHandler errorHandler() {
        // DLT Publisher — mensajes fallidos van a topic.DLT
        DeadLetterPublishingRecoverer recoverer =
            new DeadLetterPublishingRecoverer(kafkaTemplate,
                (record, ex) -> new TopicPartition(
                    record.topic() + ".DLT",
                    record.partition()
                ));

        // 3 reintentos con backoff exponencial
        ExponentialBackOffWithMaxRetries backOff =
            new ExponentialBackOffWithMaxRetries(3);
        backOff.setInitialInterval(1000L);
        backOff.setMultiplier(2.0);
        backOff.setMaxInterval(10000L);

        DefaultErrorHandler errorHandler = new DefaultErrorHandler(recoverer, backOff);

        // No reintentar estos tipos de excepciones (errores de negocio)
        errorHandler.addNotRetryableExceptions(
            BusinessException.class,
            DeserializationException.class
        );

        return errorHandler;
    }

    // Batch factory
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object>
            batchKafkaListenerContainerFactory(
            ConsumerFactory<String, Object> consumerFactory) {

        var factory = new ConcurrentKafkaListenerContainerFactory<String, Object>();
        factory.setConsumerFactory(consumerFactory);
        factory.setBatchListener(true);
        factory.setConcurrency(3);
        return factory;
    }
}
```

## Kafka Transactions

```java
@Service
@RequiredArgsConstructor
public class TransactionalOrderService {

    private final KafkaTemplate<String, Object> kafkaTemplate;
    private final OrderRepository orderRepository;

    @Transactional  // transacción JPA + Kafka
    public Order createOrder(CreateOrderRequest req) {
        Order order = orderRepository.save(new Order(req));

        // Si esto falla, el mensaje NO se envía
        kafkaTemplate.executeInTransaction(ops -> {
            ops.send("order-created", order.getId().toString(),
                new OrderCreatedEvent(order));
            return true;
        });

        return order;
    }
}
```

## Testing con EmbeddedKafka

```java
@SpringBootTest
@EmbeddedKafka(
    partitions = 1,
    topics = {"order-created", "payment-processed"},
    brokerProperties = {
        "listeners=PLAINTEXT://localhost:9092",
        "port=9092"
    }
)
class OrderEventIntegrationTest {

    @Autowired
    private EventProducer producer;

    @Autowired
    private KafkaTestUtils kafkaTestUtils;

    @Test
    void shouldPublishOrderCreatedEvent() throws Exception {
        // given
        var event = new OrderCreatedEvent(1L, 42L, BigDecimal.TEN);

        // when
        producer.send("order-created", "1", event);

        // then - verificar que llegó
        ConsumerRecord<String, String> record =
            KafkaTestUtils.getSingleRecord(consumer, "order-created");

        assertThat(record.key()).isEqualTo("1");
    }
}
```

## 📌 Notas Rápidas
- `enable-auto-commit: false` + ACK manual es la configuración recomendada en producción — el auto-commit por tiempo puede confirmar offsets de mensajes que en realidad fallaron a mitad de procesamiento.
- Un Dead Letter Topic (DLT) evita que un mensaje "envenenado" (que siempre falla) bloquee el consumo del resto de la partición reintentando infinitamente.
- `@Transactional` de Spring + `kafkaTemplate.executeInTransaction` permite unir una escritura a base de datos y un envío a Kafka como una sola unidad atómica — clave para no perder consistencia entre el estado guardado y el evento publicado.
- `EmbeddedKafka` levanta un broker en memoria para tests — alternativa más rápida a TestContainers cuando no hace falta un broker 100% real, aunque TestContainers da más fidelidad.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

