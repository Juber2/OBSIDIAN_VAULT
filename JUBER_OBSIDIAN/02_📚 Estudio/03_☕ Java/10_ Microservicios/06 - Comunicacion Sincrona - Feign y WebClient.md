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
  - OpenFeign, WebClient, RestClient, HTTP cliente, inter-service communication
---

## 🧠 Opciones para llamadas síncronas

| Cliente | Estilo | Cuándo usar |
|---------|--------|-------------|
| **OpenFeign** | Declarativo (interfaz) | Apps MVC — lo más simple |
| **WebClient** | Reactivo (fluent API) | Apps WebFlux o reactive |
| **RestClient** | Síncrono moderno (Spring 6.1+) | Reemplazo de RestTemplate |
| **RestTemplate** | Síncrono legacy | Solo en proyectos viejos |

## OpenFeign — cliente declarativo

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableFeignClients  // activar Feign
public class OrderServiceApplication { ... }
```

### Definir cliente Feign

```java
@FeignClient(
    name = "USER-SERVICE",        // nombre en Eureka
    path = "/api/usuarios",
    fallback = UserClientFallback.class
)
public interface UserClient {

    @GetMapping("/{id}")
    UsuarioResponse findById(@PathVariable Long id);

    @GetMapping
    Page<UsuarioResponse> findAll(@RequestParam int page,
                                   @RequestParam int size);

    @PostMapping
    UsuarioResponse crear(@RequestBody CrearUsuarioRequest request);

    @PutMapping("/{id}")
    UsuarioResponse actualizar(@PathVariable Long id,
                                @RequestBody ActualizarUsuarioRequest request);

    @DeleteMapping("/{id}")
    void eliminar(@PathVariable Long id);
}
```

### Fallback de Feign

```java
@Component
public class UserClientFallback implements UserClient {

    @Override
    public UsuarioResponse findById(Long id) {
        return UsuarioResponse.empty(id);
    }

    @Override
    public Page<UsuarioResponse> findAll(int page, int size) {
        return Page.empty();
    }

    @Override
    public UsuarioResponse crear(CrearUsuarioRequest request) {
        throw new ServiceUnavailableException("User Service no disponible");
    }

    // ... otros métodos
}
```

### Feign con Circuit Breaker

```yaml
spring:
  cloud:
    openfeign:
      circuitbreaker:
        enabled: true
```

### Configuración de Feign (timeout, logs)

```java
@Configuration
public class FeignConfig {

    @Bean
    public Request.Options requestOptions() {
        return new Request.Options(
            2, TimeUnit.SECONDS,    // connectTimeout
            5, TimeUnit.SECONDS,    // readTimeout
            true                    // followRedirects
        );
    }

    @Bean
    public Logger.Level feignLoggerLevel() {
        return Logger.Level.BASIC;  // NONE, BASIC, HEADERS, FULL
    }

    @Bean
    public ErrorDecoder errorDecoder() {
        return (methodKey, response) -> {
            if (response.status() == 404) {
                return new ResourceNotFoundException("Recurso no encontrado");
            }
            if (response.status() == 503) {
                return new ServiceUnavailableException("Servicio no disponible");
            }
            return new Default().decode(methodKey, response);
        };
    }
}

// Aplicar a un cliente específico
@FeignClient(name = "USER-SERVICE", configuration = FeignConfig.class)
public interface UserClient { ... }
```

### Propagar headers (ej: JWT)

```java
@Component
public class FeignAuthInterceptor implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate template) {
        // Propagar el token JWT del request actual
        ServletRequestAttributes attributes =
            (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();

        if (attributes != null) {
            String authHeader = attributes.getRequest()
                .getHeader(HttpHeaders.AUTHORIZATION);

            if (authHeader != null) {
                template.header(HttpHeaders.AUTHORIZATION, authHeader);
            }
        }
    }
}
```

## WebClient — cliente reactivo/no-bloqueante

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

```java
@Configuration
public class WebClientConfig {

    @Bean
    @LoadBalanced
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder()
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
    }
}

@Service
@RequiredArgsConstructor
public class UserWebClient {

    private final WebClient.Builder webClientBuilder;

    // GET — Mono (un elemento)
    public Mono<UsuarioResponse> getUsuario(Long id) {
        return webClientBuilder.build()
            .get()
            .uri("http://USER-SERVICE/api/usuarios/{id}", id)
            .retrieve()
            .onStatus(HttpStatusCode::is4xxClientError,
                response -> Mono.error(new ResourceNotFoundException("Usuario no encontrado")))
            .onStatus(HttpStatusCode::is5xxServerError,
                response -> Mono.error(new ServiceUnavailableException("User Service error")))
            .bodyToMono(UsuarioResponse.class)
            .timeout(Duration.ofSeconds(3))
            .retryWhen(Retry.backoff(3, Duration.ofSeconds(1)));
    }

    // GET — Flux (múltiples elementos)
    public Flux<UsuarioResponse> getAllUsuarios() {
        return webClientBuilder.build()
            .get()
            .uri("http://USER-SERVICE/api/usuarios")
            .retrieve()
            .bodyToFlux(UsuarioResponse.class);
    }

    // POST
    public Mono<UsuarioResponse> crearUsuario(CrearUsuarioRequest request) {
        return webClientBuilder.build()
            .post()
            .uri("http://USER-SERVICE/api/usuarios")
            .bodyValue(request)
            .retrieve()
            .bodyToMono(UsuarioResponse.class);
    }
}
```

## RestClient — nuevo en Spring 6.1 (Spring Boot 3.2+)

```java
@Configuration
public class RestClientConfig {

    @Bean
    public RestClient userServiceClient() {
        return RestClient.builder()
            .baseUrl("http://user-service")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();
    }
}

@Service
@RequiredArgsConstructor
public class UserRestClient {

    private final RestClient userServiceClient;

    public UsuarioResponse getUsuario(Long id) {
        return userServiceClient.get()
            .uri("/api/usuarios/{id}", id)
            .retrieve()
            .onStatus(HttpStatusCode::is4xxClientError,
                (request, response) -> {
                    throw new ResourceNotFoundException("Usuario no encontrado");
                })
            .body(UsuarioResponse.class);
    }

    public List<UsuarioResponse> getAllUsuarios() {
        return userServiceClient.get()
            .uri("/api/usuarios")
            .retrieve()
            .body(new ParameterizedTypeReference<List<UsuarioResponse>>() {});
    }
}
```

## ⚠️ Errores Comunes

> [!danger] Feign sin Circuit Breaker en producción
> Si el servicio destino se cae, Feign esperará el timeout por cada request. Siempre usar Circuit Breaker con Feign.

> [!warning] WebClient en apps bloqueantes (no reactive)
> WebClient se puede usar en apps MVC bloqueantes llamando `.block()`, pero pierde las ventajas reactivas. Usar solo para llamadas a APIs externas donde quieras non-blocking.

> [!tip] Feign vs WebClient
> En apps MVC normales, Feign es más simple y legible. WebClient es mejor para apps que necesitan concurrencia alta o ya usan WebFlux.

## 📌 Notas Rápidas
- Feign sin Circuit Breaker es la trampa más común: por default, si el servicio destino está lento (no caído), Feign espera el timeout completo en cada llamada, propagando la lentitud en cascada.
- `RestClient` (Spring 6.1+) es el reemplazo moderno recomendado de `RestTemplate` para llamadas síncronas simples — API más fluida, sin las limitaciones que llevaron a deprecar `RestTemplate` en el roadmap de Spring.
- Elegir entre comunicación síncrona (Feign/WebClient) y asíncrona (Kafka) no es solo preferencia técnica: síncrona acopla temporalmente los servicios (si uno cae, afecta al otro), asíncrona lo evita a costa de consistencia eventual — ver [[07 - Comunicacion Asincrona - Kafka]].
- Propagar el JWT del request original hacia llamadas downstream (Feign interceptor) es necesario para mantener el contexto de autorización entre microservicios — sin esto, el servicio downstream no sabe quién hizo la request original.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

