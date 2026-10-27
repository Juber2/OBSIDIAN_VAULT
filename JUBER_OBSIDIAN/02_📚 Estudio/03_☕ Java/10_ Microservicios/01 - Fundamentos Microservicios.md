---
tags:
  - Java
  - Microservicios
  - Arquitectura
Creado: 2026-05-21
Relacionado:
  - Microservicios
Palabras clave:
  - microservicios, principios, database per service, saga, eventual consistency
---

## 🧠 Principios de Microservicios

### Single Responsibility
Cada servicio hace UNA cosa bien:

```
❌ OrderService (maneja pedidos, usuarios, pagos, inventario)

✅ OrderService     → ciclo de vida del pedido
✅ UserService      → gestión de usuarios
✅ PaymentService   → procesamiento de pagos
✅ InventoryService → stock de productos
```

### Database per Service
Cada microservicio tiene su PROPIA base de datos:

```
OrderService    → PostgreSQL (orders_db)
UserService     → PostgreSQL (users_db)
ProductService  → MongoDB (products_db)
CacheService    → Redis
SearchService   → Elasticsearch
```

¿Por qué? Desacoplamiento. Si OrderService necesita datos de UserService, los pide vía API, no accede a su BD directamente.

### Comunicación

```
Síncrona  → REST (Feign/WebClient), gRPC
           → respuesta inmediata, acoplamiento temporal

Asíncrona → Kafka, RabbitMQ
           → fire-and-forget, desacoplado, más resiliente
```

## Estructura de un microservicio

```
user-service/
├── src/main/java/com/empresa/userservice/
│   ├── UserServiceApplication.java
│   ├── domain/
│   │   ├── model/          → entidades del dominio
│   │   └── repository/     → interfaces de repositorio
│   ├── application/
│   │   ├── service/        → lógica de negocio
│   │   └── dto/            → request/response
│   ├── infrastructure/
│   │   ├── persistence/    → implementaciones JPA
│   │   ├── web/            → controllers REST
│   │   └── messaging/      → producers/consumers Kafka
│   └── config/             → configuración de Spring
├── src/main/resources/
│   ├── application.yml
│   └── bootstrap.yml       → config para Config Server
└── pom.xml
```

## Problemas clásicos de microservicios

### 1. Datos distribuidos — Saga Pattern

Cuando una operación toca múltiples servicios:

```
Crear Pedido = reservar inventario + cobrar al usuario + crear pedido

Saga Coreografía (eventos):
  OrderService  → emite OrderCreated
  InventoryService → escucha, reserva stock → emite StockReserved
  PaymentService   → escucha, cobra → emite PaymentProcessed
  OrderService  → escucha, confirma pedido → emite OrderConfirmed

  Si PaymentService falla:
  PaymentService → emite PaymentFailed
  InventoryService → escucha, libera stock → emite StockReleased
  OrderService  → cancela pedido

Saga Orquestación (coordinador central):
  OrderOrchestrator coordina los pasos y compensaciones
```

### 2. Consistencia eventual

```
// En microservicios NO hay transacciones distribuidas ACID
// Se acepta que los datos estarán eventualmente consistentes

// Al crear un pedido:
Order order = orderRepository.save(new Order(userId, items));
kafkaProducer.send("order-created", new OrderCreatedEvent(order.getId(), userId));
// UserService actualizará su historial cuando procese el evento
// Por un momento los datos son inconsistentes → eventualmente consistentes
```

### 3. Idempotencia

```java
// Los mensajes pueden llegar duplicados → los handlers deben ser idempotentes
@KafkaListener(topics = "payment-processed")
public void handlePaymentProcessed(PaymentProcessedEvent event) {
    // Verificar si ya se procesó este evento
    if (orderRepository.existsByPaymentId(event.getPaymentId())) {
        log.warn("Evento duplicado ignorado: {}", event.getPaymentId());
        return;
    }
    // procesar...
}
```

## Patrones de diseño para microservicios

| Patrón | Problema que resuelve |
|--------|----------------------|
| **API Gateway** | Punto único de entrada, routing, auth |
| **Circuit Breaker** | Fallos en cascada |
| **Saga** | Transacciones distribuidas |
| **CQRS** | Separar lecturas de escrituras |
| **Event Sourcing** | Auditoría completa de cambios |
| **Outbox Pattern** | Garantizar envío de eventos Kafka |
| **Strangler Fig** | Migrar monolito a microservicios gradualmente |

## Outbox Pattern — garantizar consistencia

```java
// Problema: guardar en BD Y enviar a Kafka → si falla Kafka, dato guardado sin evento

// Solución: guardar evento en tabla outbox dentro de la MISMA transacción
@Transactional
public Order createOrder(CreateOrderRequest req) {
    Order order = orderRepository.save(new Order(req));

    // Guardar evento en BD en la MISMA transacción
    OutboxEvent event = new OutboxEvent(
        "ORDER_CREATED",
        objectMapper.writeValueAsString(new OrderCreatedEvent(order))
    );
    outboxRepository.save(event);

    return order;
}

// Worker separado lee la tabla outbox y publica en Kafka
@Scheduled(fixedDelay = 1000)
public void processOutboxEvents() {
    List<OutboxEvent> events = outboxRepository.findUnprocessed();
    for (OutboxEvent event : events) {
        kafkaProducer.send(event.getTopic(), event.getPayload());
        event.setProcessed(true);
        outboxRepository.save(event);
    }
}
```

## 📌 Notas Rápidas
- "Database per service" es lo que hace inevitable la consistencia eventual y el patrón SAGA — no es una elección arbitraria, es consecuencia directa de dar autonomía real a cada servicio (ver [[ARQUITECTURA]]).
- El Outbox Pattern resuelve un problema muy concreto: sin él, un fallo entre "guardar en BD" y "publicar en Kafka" deja el sistema inconsistente sin forma de detectarlo — con outbox, ambas cosas ocurren en la misma transacción local.
- Idempotencia en los consumers no es opcional en sistemas con colas/eventos — los brokers garantizan "at-least-once" en la práctica, así que los duplicados van a pasar tarde o temprano.
- Antes de partir un monolito en microservicios, vale preguntarse si el problema real es de escala de equipo/despliegue (donde microservicios ayuda) o de mal diseño interno (donde Hexagonal/Clean dentro del mismo monolito puede alcanzar sin el costo operacional extra).

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

