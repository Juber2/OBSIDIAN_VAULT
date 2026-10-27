---
tags:
  - Java
  - Spring
  - Microservicios
Creado: 2026-05-21
Relacionado:
  - Spring Cloud
  - Microservicios
Palabras clave:
  - Eureka, service discovery, registro, load balancing, instancias
---

## 🧠 ¿Por qué Service Discovery?

En microservicios, las instancias pueden estar en IPs/puertos dinámicos (Docker, K8s). El Service Discovery resuelve:

```
Sin Service Discovery:
  order-service → http://192.168.1.10:8081/users/123  ← IP hardcodeada ❌

Con Eureka:
  order-service → http://USER-SERVICE/users/123  ← nombre lógico ✅
  Eureka resuelve la IP/puerto actual automáticamente
```

## Eureka Server

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml del Eureka Server
server:
  port: 8761

spring:
  application:
    name: eureka-server

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false  # el server no se registra a sí mismo
    fetch-registry: false
  server:
    enable-self-preservation: false  # útil en desarrollo
    eviction-interval-timer-in-ms: 5000
```

## Eureka Client (cada microservicio)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```java
@SpringBootApplication
// @EnableDiscoveryClient  → opcional en Spring Boot 3, auto-detectado
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

```yaml
# application.yml del microservicio
spring:
  application:
    name: order-service  # ← nombre con el que se registra en Eureka

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    register-with-eureka: true
    fetch-registry: true
    registry-fetch-interval-seconds: 10
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10  # heartbeat cada 10s
    lease-expiration-duration-in-seconds: 30  # eliminar si no hay heartbeat en 30s
    instance-id: ${spring.application.name}:${random.uuid}
```

## Consultar Eureka programáticamente

```java
@Service
@RequiredArgsConstructor
public class ServiceDiscoveryService {

    private final DiscoveryClient discoveryClient;

    public List<ServiceInstance> getInstancias(String serviceName) {
        return discoveryClient.getInstances(serviceName);
    }

    public String getUrlServicio(String serviceName) {
        List<ServiceInstance> instancias = discoveryClient.getInstances(serviceName);
        if (instancias.isEmpty()) {
            throw new RuntimeException("Servicio no disponible: " + serviceName);
        }
        // round-robin simple
        ServiceInstance instancia = instancias.get(0);
        return instancia.getUri().toString();
    }

    public List<String> getServiciosRegistrados() {
        return discoveryClient.getServices();
    }
}
```

## Load Balancing con RestTemplate (@LoadBalanced)

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced  // Spring Cloud LoadBalancer resuelve el nombre del servicio
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
@RequiredArgsConstructor
public class UserClient {

    private final RestTemplate restTemplate;

    public UsuarioResponse getUsuario(Long id) {
        // USER-SERVICE → nombre registrado en Eureka (en mayúsculas)
        return restTemplate.getForObject(
            "http://USER-SERVICE/api/usuarios/" + id,
            UsuarioResponse.class);
    }
}
```

## Load Balancing con WebClient (@LoadBalanced)

```java
@Configuration
public class WebClientConfig {

    @Bean
    @LoadBalanced
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder();
    }
}

@Service
@RequiredArgsConstructor
public class UserClient {

    private final WebClient.Builder webClientBuilder;

    public Mono<UsuarioResponse> getUsuario(Long id) {
        return webClientBuilder.build()
            .get()
            .uri("http://USER-SERVICE/api/usuarios/" + id)
            .retrieve()
            .bodyToMono(UsuarioResponse.class);
    }
}
```

## Eureka en producción — Alta disponibilidad

```yaml
# Eureka 1 (peer awareness)
eureka:
  instance:
    hostname: eureka1
  client:
    service-url:
      defaultZone: http://eureka2:8761/eureka/,http://eureka3:8761/eureka/

# Los clientes se conectan a todos:
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:8761/eureka/,http://eureka2:8761/eureka/
```

## Alternativas a Eureka

| Herramienta | Observación |
|-------------|------------|
| **Eureka** | Simple, parte de Spring Cloud, bueno para Java |
| **Consul** | Más features (KV store, health checks, DNS) |
| **Kubernetes** | En K8s, el discovery es nativo — Eureka no necesario |
| **Zookeeper** | Más complejo, usado con Kafka |

> [!tip] En Kubernetes no necesitas Eureka
> K8s tiene su propio Service Discovery. Con K8s, solo configura los Service de K8s y usa los nombres de los Services como hostnames.

## 📌 Notas Rápidas
- Este es un punto de entrevista importante para el puesto que estás preparando: en Kubernetes/AKS **no hace falta Eureka** — el Service de Kubernetes ya resuelve el nombre lógico a IPs actuales vía DNS interno (ver [[03 - Services y Networking]]). Traer Eureka a un cluster K8s es duplicar una capa que la plataforma ya resuelve.
- `@LoadBalanced` es lo que hace que `RestTemplate`/`WebClient` interpreten `http://USER-SERVICE/...` como un nombre lógico a resolver, en vez de un hostname DNS real.
- El "self-preservation mode" de Eureka evita que el registro completo se vacíe de golpe ante una partición de red transitoria — desactivarlo en dev simplifica el comportamiento, pero mantenerlo activo en producción evita falsos positivos masivos.
- `lease-expiration-duration` (cuánto tarda en dar de baja una instancia sin heartbeat) es un trade-off: muy corto genera bajas prematuras por latencia momentánea, muy largo deja tráfico yendo a instancias caídas.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

