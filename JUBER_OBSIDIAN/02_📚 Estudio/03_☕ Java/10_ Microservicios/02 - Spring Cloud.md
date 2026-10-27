---
tags:
  - Java
  - Spring
  - Microservicios
Creado: 2026-05-21
Relacionado:
  - Microservicios
  - Spring Boot
Palabras clave:
  - Spring Cloud, starters, bootstrap, spring-cloud-dependencies
---

## 🧠 ¿Qué es Spring Cloud?

Conjunto de herramientas para construir sistemas distribuidos y microservicios con Spring Boot.

## Componentes principales

| Componente | Propósito |
|------------|-----------|
| **Spring Cloud Gateway** | API Gateway — routing y filtros |
| **Spring Cloud Netflix Eureka** | Service Discovery |
| **Spring Cloud Config** | Configuración centralizada |
| **Spring Cloud LoadBalancer** | Balanceo de carga del lado del cliente |
| **Spring Cloud OpenFeign** | Cliente HTTP declarativo |
| **Resilience4j** | Circuit Breaker, Retry, Rate Limiter |
| **Micrometer Tracing** | Distributed Tracing |
| **Spring Cloud Bus** | Propagar cambios de config |

## Dependencias base (parent POM)

```xml
<!-- pom.xml del microservicio -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.0</version>
</parent>

<properties>
    <spring-cloud.version>2023.0.3</spring-cloud.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Registro y descubrimiento con Eureka -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <!-- Config Server client -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>

    <!-- Feign para llamadas a otros servicios -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>

    <!-- Circuit Breaker -->
    <dependency>
        <groupId>io.github.resilience4j</groupId>
        <artifactId>resilience4j-spring-boot3</artifactId>
    </dependency>

    <!-- Load Balancer -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>
</dependencies>
```

## application.yml del microservicio

```yaml
spring:
  application:
    name: order-service   # nombre del servicio en Eureka

  config:
    import: optional:configserver:http://localhost:8888  # Config Server

  cloud:
    config:
      fail-fast: true      # fallar si no se puede conectar al Config Server

server:
  port: 8082

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10
    lease-expiration-duration-in-seconds: 30
```

## Estructura multi-módulo Maven

```
parent-pom/
├── pom.xml  (parent con Spring Cloud BOM)
├── eureka-server/
│   └── pom.xml
├── api-gateway/
│   └── pom.xml
├── config-server/
│   └── pom.xml
├── user-service/
│   └── pom.xml
├── order-service/
│   └── pom.xml
└── payment-service/
    └── pom.xml
```

```xml
<!-- parent pom.xml -->
<groupId>com.empresa</groupId>
<artifactId>microservicios-parent</artifactId>
<version>1.0.0</version>
<packaging>pom</packaging>

<modules>
    <module>eureka-server</module>
    <module>api-gateway</module>
    <module>config-server</module>
    <module>user-service</module>
    <module>order-service</module>
    <module>payment-service</module>
</modules>
```

## Health checks y readiness

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics
  endpoint:
    health:
      show-details: always
  health:
    circuitbreakers:
      enabled: true
    ratelimiters:
      enabled: true
```

## Compatibilidad de versiones

| Spring Boot | Spring Cloud |
|-------------|-------------|
| 3.3.x | 2023.0.x |
| 3.2.x | 2023.0.x |
| 3.1.x | 2022.0.x |
| 3.0.x | 2022.0.x |

> [!warning] Siempre usar el BOM `spring-cloud-dependencies`
> No definir versiones individuales de cada módulo Spring Cloud. El BOM asegura compatibilidad entre todos los módulos.

## 📌 Notas Rápidas
- Spring Cloud es un paraguas de proyectos independientes (Gateway, Eureka, Config, OpenFeign) — no hace falta usarlos todos juntos, se eligen según lo que el sistema realmente necesita.
- El BOM `spring-cloud-dependencies` resuelve el problema de compatibilidad entre módulos — mezclar versiones sueltas de cada componente es una fuente clásica de errores difíciles de diagnosticar.
- En Kubernetes, buena parte de lo que resuelve Spring Cloud (Service Discovery, Config, Load Balancing) ya lo da la plataforma nativamente — vale la pena evaluar si conviene duplicar esa responsabilidad en la capa de aplicación.
- `fail-fast: true` en la config del Config Server es preferible en producción — mejor que la app no arranque a que arranque con configuración incompleta/desactualizada.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

