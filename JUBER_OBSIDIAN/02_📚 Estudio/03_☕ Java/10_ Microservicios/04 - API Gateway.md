---
tags:
  - Java
  - Spring
  - Microservicios
Creado: 2026-05-21
Relacionado:
  - Spring Cloud
  - Microservicios
  - Spring Security
Palabras clave:
  - API Gateway, Spring Cloud Gateway, routing, filtros, rate limiting
---

## 🧠 ¿Qué hace un API Gateway?

Punto único de entrada para todos los microservicios:

```
Cliente → API Gateway → user-service
                     → order-service
                     → payment-service
```

Responsabilidades:
- Routing
- Autenticación / Autorización JWT
- Rate Limiting
- Logging / Tracing
- Transformación de requests/responses
- SSL termination
- Load Balancing

## Dependencia

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!-- Para load balancing con Eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

> [!warning] Spring Cloud Gateway es reactivo (WebFlux)
> NO combinar con spring-boot-starter-web. Usa spring-boot-starter-webflux internamente.

## Configuración de rutas (YAML)

```yaml
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # Ruta al User Service
        - id: user-service
          uri: lb://USER-SERVICE       # lb:// → load balanced via Eureka
          predicates:
            - Path=/api/usuarios/**
          filters:
            - StripPrefix=0            # no eliminar prefijo
            - AddRequestHeader=X-Gateway-Source, gateway

        # Ruta al Order Service
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/pedidos/**
            - Method=GET,POST,PUT,DELETE

        # Ruta con rewrite
        - id: product-service
          uri: lb://PRODUCT-SERVICE
          predicates:
            - Path=/productos/**
          filters:
            - RewritePath=/productos/(?<segment>.*), /api/productos/${segment}

        # Ruta estática (sin Eureka)
        - id: external-api
          uri: https://api.externa.com
          predicates:
            - Path=/externo/**
          filters:
            - StripPrefix=1

      # Configuración global
      default-filters:
        - DedupeResponseHeader=Access-Control-Allow-Origin
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "http://localhost:3000"
            allowedMethods: "*"
            allowedHeaders: "*"
```

## Configuración de rutas (Java DSL)

```java
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator routeLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r
                .path("/api/usuarios/**")
                .filters(f -> f
                    .addRequestHeader("X-Gateway-Source", "gateway")
                    .addResponseHeader("X-Response-Time", String.valueOf(System.currentTimeMillis()))
                )
                .uri("lb://USER-SERVICE")
            )
            .route("order-service", r -> r
                .path("/api/pedidos/**")
                .and()
                .method(HttpMethod.GET, HttpMethod.POST)
                .uri("lb://ORDER-SERVICE")
            )
            .build();
    }
}
```

## Filtro global de autenticación JWT

```java
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter implements GlobalFilter, Ordered {

    private final JwtService jwtService;

    private static final List<String> PUBLIC_PATHS = List.of(
        "/api/auth/login",
        "/api/auth/register",
        "/actuator"
    );

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String path = exchange.getRequest().getURI().getPath();

        // Rutas públicas — pasar sin validar
        if (PUBLIC_PATHS.stream().anyMatch(path::startsWith)) {
            return chain.filter(exchange);
        }

        String authHeader = exchange.getRequest()
            .getHeaders().getFirst(HttpHeaders.AUTHORIZATION);

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            return onError(exchange, HttpStatus.UNAUTHORIZED);
        }

        String token = authHeader.substring(7);

        try {
            String username = jwtService.extractUsername(token);
            String roles = jwtService.extractRoles(token);

            // Propagar info al microservicio downstream
            ServerHttpRequest mutatedRequest = exchange.getRequest().mutate()
                .header("X-User-Email", username)
                .header("X-User-Roles", roles)
                .build();

            return chain.filter(exchange.mutate().request(mutatedRequest).build());

        } catch (JwtException e) {
            return onError(exchange, HttpStatus.UNAUTHORIZED);
        }
    }

    private Mono<Void> onError(ServerWebExchange exchange, HttpStatus status) {
        exchange.getResponse().setStatusCode(status);
        return exchange.getResponse().setComplete();
    }

    @Override
    public int getOrder() {
        return -1;  // ejecutar primero
    }
}
```

## Rate Limiting

```xml
<!-- Requiere Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/api/usuarios/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10    # tokens por segundo
                redis-rate-limiter.burstCapacity: 20    # máximo en burst
                key-resolver: "#{@ipKeyResolver}"       # agrupar por IP
```

```java
@Configuration
public class RateLimiterConfig {

    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest().getRemoteAddress().getAddress().getHostAddress()
        );
    }

    // Por usuario autenticado
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.justOrEmpty(
            exchange.getRequest().getHeaders().getFirst("X-User-Email")
        ).defaultIfEmpty("anonymous");
    }
}
```

## Circuit Breaker en el Gateway

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/pedidos/**
          filters:
            - name: CircuitBreaker
              args:
                name: orderServiceCB
                fallbackUri: forward:/fallback/orders
```

```java
@RestController
public class FallbackController {

    @GetMapping("/fallback/orders")
    public ResponseEntity<Map<String, String>> ordersFallback() {
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
            .body(Map.of(
                "error", "El servicio de pedidos no está disponible",
                "message", "Por favor intente más tarde"
            ));
    }
}
```

## Logging de requests en el Gateway

```java
@Component
public class LoggingFilter implements GlobalFilter, Ordered {

    private static final Logger log = LoggerFactory.getLogger(LoggingFilter.class);

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        log.info("→ {} {} [{}]",
            request.getMethod(),
            request.getURI().getPath(),
            request.getRemoteAddress());

        long startTime = System.currentTimeMillis();

        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            long duration = System.currentTimeMillis() - startTime;
            log.info("← {} {} {}ms",
                exchange.getResponse().getStatusCode(),
                request.getURI().getPath(),
                duration);
        }));
    }

    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE;
    }
}
```

## 📌 Notas Rápidas
- Spring Cloud Gateway es reactivo (WebFlux) por diseño — necesita alta concurrencia con I/O no bloqueante porque literalmente todo el tráfico de la plataforma pasa por él.
- En un contexto Azure/AKS, esta responsabilidad se puede resolver con Application Gateway + AGIC en vez de Spring Cloud Gateway — ver [[04 - APIM y Application Gateway]] y [[05 - Ingress y Exposicion Externa]]; la elección depende de si se quiere lógica de gateway en código Java o delegada a un servicio administrado de Azure.
- Propagar el usuario autenticado vía headers (`X-User-Email`, `X-User-Roles`) hacia los microservicios downstream es un patrón común — evita que cada microservicio tenga que validar el JWT completo de nuevo, confiando en que el Gateway ya lo hizo.
- Rate limiting en el Gateway (vs en cada microservicio) centraliza la protección contra abuso en un solo punto, coherente con por qué existe un Gateway en primer lugar: no duplicar cross-cutting concerns en cada servicio.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

