---
tags:
  - Java
  - Spring Boot
Creado: 2026-05-21
Relacionado:
  - Spring Boot
Palabras clave:
  - Actuator, health, metrics, info, endpoints, Prometheus, Micrometer
---

## 🧠 Concepto

**Spring Boot Actuator** expone endpoints HTTP para monitorear y gestionar la aplicación en producción: health checks, métricas, info, etc.

## Dependencia

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

## Configuración básica

```properties
# Exponer todos los endpoints (en prod, solo exponer los necesarios)
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.include=health,info,metrics,prometheus

# Mostrar detalles de health
management.endpoint.health.show-details=always
management.endpoint.health.show-components=always

# Base path (default: /actuator)
management.endpoints.web.base-path=/management

# Puerto diferente para Actuator (buena práctica en prod)
management.server.port=8081
```

## Endpoints principales

| Endpoint | Descripción |
|----------|-------------|
| `/actuator/health` | Estado de la app y sus dependencias |
| `/actuator/info` | Info de la aplicación |
| `/actuator/metrics` | Métricas disponibles |
| `/actuator/metrics/{nombre}` | Métrica específica |
| `/actuator/env` | Variables de entorno y propiedades |
| `/actuator/loggers` | Niveles de log (puede cambiar en runtime) |
| `/actuator/beans` | Todos los beans registrados |
| `/actuator/mappings` | Todos los endpoints REST |
| `/actuator/threaddump` | Dump de threads |
| `/actuator/heapdump` | Dump del heap |
| `/actuator/shutdown` | Apagar la app (desactivado por defecto) |

## Health checks

```
GET /actuator/health
{
  "status": "UP",
  "components": {
    "db": { "status": "UP", "details": { "database": "PostgreSQL" } },
    "diskSpace": { "status": "UP", "details": { "free": "50GB" } },
    "redis": { "status": "UP" }
  }
}
```

### Health check personalizado

```java
@Component
public class ApiExternaHealthIndicator implements HealthIndicator {

    @Autowired
    private ApiExternaClient client;

    @Override
    public Health health() {
        try {
            client.ping();
            return Health.up()
                .withDetail("api", "Disponible")
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```

## Info endpoint

```properties
# application.properties
info.app.nombre=Mi Aplicación
info.app.version=@project.version@  # desde pom.xml
info.app.descripcion=API de gestión de usuarios
```

```java
// O programáticamente
@Component
public class AppInfoContributor implements InfoContributor {
    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("startup", LocalDateTime.now().toString());
        builder.withDetail("jvm.version", System.getProperty("java.version"));
    }
}
```

## Métricas con Micrometer + Prometheus

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```java
@Service
public class PedidoService {
    private final Counter pedidosCreados;
    private final Timer tiempoProcesamiento;

    public PedidoService(MeterRegistry registry) {
        this.pedidosCreados = Counter.builder("pedidos.creados")
            .description("Total pedidos creados")
            .tag("tipo", "online")
            .register(registry);

        this.tiempoProcesamiento = Timer.builder("pedidos.procesamiento")
            .description("Tiempo de procesamiento")
            .register(registry);
    }

    public Pedido crear(PedidoRequest request) {
        return tiempoProcesamiento.record(() -> {
            Pedido pedido = procesarPedido(request);
            pedidosCreados.increment();
            return pedido;
        });
    }
}
```

## 📌 Notas Rápidas
