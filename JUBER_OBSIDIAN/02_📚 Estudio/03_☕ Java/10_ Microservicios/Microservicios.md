---
tags:
  - Java
  - Spring
  - Microservicios
  - Arquitectura
Creado: 2026-05-21
Relacionado:
  - Spring Boot
  - Spring Cloud
  - Kafka
Palabras clave:
  - microservicios, Spring Cloud, Eureka, Gateway, Circuit Breaker, Feign
---

#Principal

# ☁️ Microservicios con Java

## Progreso

| Tema | Estado |
|------|--------|
| [[01 - Fundamentos Microservicios]] | ⬜ |
| [[02 - Spring Cloud]] | ⬜ |
| [[03 - Service Discovery - Eureka]] | ⬜ |
| [[04 - API Gateway]] | ⬜ |
| [[05 - Circuit Breaker - Resilience4j]] | ⬜ |
| [[06 - Comunicacion Sincrona - Feign y WebClient]] | ⬜ |
| [[07 - Comunicacion Asincrona - Kafka]] | ⬜ |
| [[08 - Distributed Tracing]] | ⬜ |
| [[09 - Config Server]] | ⬜ |
| [[10 - Docker y Kubernetes para Microservicios]] | ⬜ |

## 🧠 Monolito vs Microservicios

| | Monolito | Microservicios |
|-|----------|----------------|
| Deploy | Todo junto | Independiente por servicio |
| Escala | Toda la app | Solo el servicio que lo necesita |
| Tecnología | Una sola | Cada servicio puede tener la suya |
| Complejidad | Baja al inicio | Alta (red, datos distribuidos) |
| Fallo | Todo o nada | Aislado por servicio |
| Ideal para | Equipos pequeños, MVP | Equipos grandes, alta escala |

## Stack típico con Spring Cloud

```
[Cliente]
    ↓
[API Gateway] (Spring Cloud Gateway)
    ↓
[Service Discovery] (Eureka)
    ↓
[Microservicio A] ←→ [Microservicio B]  (Feign / WebClient)
    ↓                       ↓
[Base de datos A]   [Base de datos B]    (BD por servicio)
    ↓
[Message Broker] (Kafka / RabbitMQ)     (comunicación async)
    ↓
[Config Server]                          (configuración centralizada)
    ↓
[Distributed Tracing] (Micrometer + Zipkin/Tempo)
```

## Versiones Spring Cloud (2024+)

```xml
<!-- BOM para gestionar versiones -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2023.0.3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
