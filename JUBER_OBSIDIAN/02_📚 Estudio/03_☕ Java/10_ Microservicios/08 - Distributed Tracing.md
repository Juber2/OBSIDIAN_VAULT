---
tags:
  - Java
  - Spring
  - Microservicios
  - Observabilidad
Creado: 2026-05-21
Relacionado:
  - Microservicios
  - Spring Boot Actuator
Palabras clave:
  - tracing, Zipkin, Jaeger, Tempo, TraceId, SpanId, Micrometer, correlación
---

## 🧠 ¿Por qué Distributed Tracing?

```
Request del usuario → Gateway → OrderService → UserService → PaymentService

Sin tracing: "¿por qué tardó 3 segundos?" → imposible saber dónde
Con tracing: ver la traza completa con tiempos por servicio y span
```

## Conceptos

| Concepto | Descripción |
|----------|-------------|
| **Trace** | La request completa de punta a punta (un traceId) |
| **Span** | Una operación dentro de la traza (un servicio, una BD query) |
| **TraceId** | ID único que identifica la traza completa |
| **SpanId** | ID del span actual |
| **ParentSpanId** | ID del span padre |
| **Baggage** | Datos que viajan con el trace (userId, tenantId) |

## Dependencias (Micrometer Tracing + Zipkin)

```xml
<!-- Micrometer Tracing -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>

<!-- Exportar a Zipkin -->
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>

<!-- Para propagar por Kafka -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing</artifactId>
</dependency>
```

## Configuración

```yaml
management:
  tracing:
    sampling:
      probability: 1.0     # 1.0 = 100% de requests trackeadas (0.1 en producción)
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans

logging:
  pattern:
    # Incluir traceId y spanId en todos los logs
    level: "%5p [${spring.application.name},%X{traceId},%X{spanId}]"
```

## Logs con traceId automático

```
# Sin tracing:
2024-01-01 INFO OrderService - Procesando pedido 123

# Con Micrometer Tracing:
2024-01-01 INFO [order-service,65bf4f8a3c5a9b3e,65bf4f8a3c5a9b3e] - Procesando pedido 123
2024-01-01 INFO [user-service,65bf4f8a3c5a9b3e,a3e8f5b2c9d1e7f0] - Buscando usuario 42
2024-01-01 INFO [payment-service,65bf4f8a3c5a9b3e,c7f9a2b8d3e5f1a4] - Cobrando 99.99
```

El mismo `traceId` (`65bf4f8a3c5a9b3e`) atraviesa los 3 servicios.

## Spans personalizados

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final Tracer tracer;

    public Order createOrder(CreateOrderRequest req) {
        // Span para operación de negocio específica
        Span span = tracer.nextSpan().name("create-order").start();

        try (Tracer.SpanInScope ws = tracer.withSpan(span)) {
            span.tag("userId", req.userId().toString());
            span.tag("itemCount", String.valueOf(req.items().size()));

            Order order = processOrder(req);

            span.tag("orderId", order.getId().toString());
            return order;

        } catch (Exception ex) {
            span.error(ex);
            throw ex;
        } finally {
            span.end();
        }
    }
}
```

## @NewSpan — con AOP

```java
@Service
public class OrderService {

    @NewSpan("validate-inventory")  // crea un span nuevo
    public void validateInventory(List<OrderItem> items) {
        // span automático para este método
    }

    @ContinueSpan(log = "process-payment")  // continúa el span actual con log
    public void processPayment(PaymentRequest req) {
        // ...
    }
}
```

## Baggage — propagar datos en el trace

```java
@Component
@RequiredArgsConstructor
public class TenantBaggageFilter implements Filter {

    private final BaggageManager baggageManager;

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        String tenantId = ((HttpServletRequest) req).getHeader("X-Tenant-Id");

        if (tenantId != null) {
            // El tenantId viaja con el trace a todos los servicios downstream
            BaggageField.create("tenantId").updateValue(tenantId);
        }

        chain.doFilter(req, res);
    }
}

// Leer el baggage en cualquier parte
String tenantId = BaggageField.create("tenantId").getValue();
```

## Zipkin — Docker Compose

```yaml
# docker-compose.yml
services:
  zipkin:
    image: openzipkin/zipkin:latest
    ports:
      - "9411:9411"
```

```
Acceder a: http://localhost:9411
→ ver trazas
→ filtrar por servicio, duración, tags
→ ver la cascada de spans con tiempos
```

## Grafana Tempo (alternativa a Zipkin — más usada en producción)

```yaml
management:
  tracing:
    sampling:
      probability: 1.0
  otlp:
    tracing:
      endpoint: http://localhost:4318/v1/traces   # OTLP HTTP
```

```yaml
# Grafana Agent / Alloy config para recolectar trazas
# + Grafana Dashboard para correlacionar logs (Loki) + métricas (Prometheus) + trazas (Tempo)
```

## Correlacionar logs con traceId en Loki/CloudWatch

```yaml
logging:
  pattern:
    level: "%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]"
```

```java
// Buscar en logs por traceId:
// "traceId=65bf4f8a3c5a9b3e" → ver todos los logs de esa request
```

## 📌 Notas Rápidas
