---
tags:
  - Java
  - Spring
  - Microservicios
Creado: 2026-05-21
Relacionado:
  - Microservicios
  - Spring Cloud
Palabras clave:
  - Circuit Breaker, Resilience4j, Retry, Rate Limiter, Bulkhead, fallback
---

## 🧠 ¿Por qué Circuit Breaker?

```
Sin Circuit Breaker:
  OrderService → llama a PaymentService (caído)
  → espera 30s (timeout)
  → falla → usuario espera 30s
  → colas se llenan → OrderService también colapsa
  → efecto en cascada (cascade failure)

Con Circuit Breaker:
  OrderService → PaymentService (caído)
  → CB detecta fallos repetidos → abre el circuito
  → fallos futuros devuelven fallback inmediato
  → OrderService sigue funcionando degradado
```

## Estados del Circuit Breaker

```
CLOSED (normal) ──── fallos > umbral ──→ OPEN
   ↑                                        │
   └─── éxito ──── HALF_OPEN ←── timeout ──┘
                   (prueba con pocas llamadas)
```

## Dependencia

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## Configuración (application.yml)

```yaml
resilience4j:
  circuitbreaker:
    instances:
      payment-service:
        registerHealthIndicator: true
        slidingWindowType: COUNT_BASED       # o TIME_BASED
        slidingWindowSize: 10               # últimas 10 llamadas
        minimumNumberOfCalls: 5             # mínimo antes de abrir
        failureRateThreshold: 50            # 50% de fallos → abre
        waitDurationInOpenState: 10s        # esperar 10s antes de HALF_OPEN
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
        recordExceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException
          - feign.FeignException
        ignoreExceptions:
          - com.empresa.exception.BusinessException

  retry:
    instances:
      payment-service:
        maxAttempts: 3
        waitDuration: 1s
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2     # 1s, 2s, 4s
        retryExceptions:
          - java.io.IOException
        ignoreExceptions:
          - com.empresa.exception.BusinessException

  timelimiter:
    instances:
      payment-service:
        timeoutDuration: 3s
        cancelRunningFuture: true

  ratelimiter:
    instances:
      payment-service:
        limitForPeriod: 10                  # 10 llamadas
        limitRefreshPeriod: 1s              # por segundo
        timeoutDuration: 500ms

  bulkhead:
    instances:
      payment-service:
        maxConcurrentCalls: 10              # máx 10 llamadas concurrentes
        maxWaitDuration: 100ms
```

## @CircuitBreaker — uso en servicio

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final PaymentClient paymentClient;

    @CircuitBreaker(name = "payment-service", fallbackMethod = "fallbackPago")
    @Retry(name = "payment-service")
    @TimeLimiter(name = "payment-service")
    public CompletableFuture<PagoResponse> procesarPago(PagoRequest request) {
        return CompletableFuture.supplyAsync(() ->
            paymentClient.procesar(request));
    }

    // Fallback — firma debe coincidir con el método principal + Throwable
    public CompletableFuture<PagoResponse> fallbackPago(PagoRequest request,
                                                          Throwable ex) {
        log.error("Fallback activado para pago: {}", ex.getMessage());
        return CompletableFuture.completedFuture(
            new PagoResponse(null, "PENDIENTE", "Pago en cola para procesar")
        );
    }
}
```

## Múltiples fallbacks (por tipo de excepción)

```java
@CircuitBreaker(name = "user-service", fallbackMethod = "fallbackUsuario")
public UsuarioResponse getUsuario(Long id) {
    return userClient.findById(id);
}

// Fallback para CallNotPermittedException (circuito abierto)
public UsuarioResponse fallbackUsuario(Long id, CallNotPermittedException ex) {
    return UsuarioResponse.empty("Servicio temporalmente no disponible");
}

// Fallback genérico (cualquier otra excepción)
public UsuarioResponse fallbackUsuario(Long id, Exception ex) {
    return UsuarioResponse.empty("Error al obtener usuario");
}
```

## @Bulkhead — limitar concurrencia

```java
// Semaphore Bulkhead (síncrono)
@Bulkhead(name = "payment-service", fallbackMethod = "fallbackBulkhead")
public PagoResponse procesarPago(PagoRequest request) {
    return paymentClient.procesar(request);
}

// Thread Pool Bulkhead (asíncrono)
@Bulkhead(name = "payment-service",
          type = Bulkhead.Type.THREADPOOL,
          fallbackMethod = "fallbackBulkhead")
public CompletableFuture<PagoResponse> procesarPagoAsync(PagoRequest request) {
    return CompletableFuture.supplyAsync(() -> paymentClient.procesar(request));
}

public PagoResponse fallbackBulkhead(PagoRequest request, BulkheadFullException ex) {
    throw new ServiceUnavailableException("Sistema sobrecargado, intente más tarde");
}
```

## Monitoreo del Circuit Breaker

```java
@Component
@RequiredArgsConstructor
public class CircuitBreakerMonitor {

    private final CircuitBreakerRegistry circuitBreakerRegistry;

    @EventListener
    public void onApplicationReady(ApplicationReadyEvent event) {
        circuitBreakerRegistry.getAllCircuitBreakers()
            .forEach(cb -> {
                cb.getEventPublisher()
                    .onStateTransition(e ->
                        log.warn("CB {} → {}", cb.getName(), e.getStateTransition()))
                    .onFailureRateExceeded(e ->
                        log.error("Failure rate: {}%", e.getFailureRate()));
            });
    }
}
```

## Actuator endpoints

```yaml
management:
  health:
    circuitbreakers:
      enabled: true
  endpoints:
    web:
      exposure:
        include: health, circuitbreakers, retries
```

```bash
# Ver estado del circuit breaker
GET /actuator/health
# Respuesta:
{
  "status": "UP",
  "components": {
    "circuitBreakers": {
      "status": "UP",
      "details": {
        "payment-service": {
          "status": "CIRCUIT_CLOSED",
          "details": {
            "failureRate": "0.0%",
            "state": "CLOSED"
          }
        }
      }
    }
  }
}
```

## Fallback vs Compensación

> [!tip] Fallback no es siempre la respuesta correcta
> - **Fallback**: devuelve una respuesta degradada (caché, default, mensaje)
> - **Compensación**: deshace operaciones ya realizadas (saga pattern)
> 
> Ejemplo: si el pago falla, no basta con un fallback — hay que compensar (no crear el pedido o cancelarlo).

## 📌 Notas Rápidas
- Circuit Breaker es en el fondo una variante de State pattern (GoF): CLOSED/OPEN/HALF_OPEN son estados con transiciones bien definidas.
- Resilience4j reemplazó a Hystrix (deprecado por Netflix) como estándar de facto en el ecosistema Spring/Java para resiliencia.
- Combinar Circuit Breaker + Retry + Timeout + Bulkhead es lo normal en producción — cada uno protege contra un problema distinto (fallos repetidos, lentitud, sobrecarga, saturación de recursos).
- No es lo mismo que compensación (SAGA): el Circuit Breaker evita seguir golpeando un servicio caído; la compensación deshace efectos ya aplicados cuando un paso posterior falla — ver [[ARQUITECTURA]].

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

