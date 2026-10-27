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
  - Config Server, configuración centralizada, refresh, Spring Cloud Config, Vault
---

## 🧠 ¿Por qué Config Server?

Sin Config Server: cada microservicio tiene su `application.yml` → cambiar una propiedad = redeploy de todos los servicios.

Con Config Server: configuración centralizada en Git → cambio sin redeploy.

```
Config Server → lee desde Git → sirve config a microservicios
             → o desde: filesystem, Vault, AWS SSM
```

## Config Server

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

```yaml
# application.yml del Config Server
server:
  port: 8888

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/empresa/config-repo
          # O local para desarrollo:
          # uri: file:///C:/dev/config-repo
          default-label: main
          search-paths:
            - "{application}"   # busca en carpeta con nombre del servicio
          clone-on-start: true
          timeout: 4
          # Para repo privado:
          username: ${GIT_USERNAME}
          password: ${GIT_TOKEN}
```

## Repositorio Git de configuración

```
config-repo/
├── application.yml            # común a todos los servicios
├── order-service.yml          # específico de order-service
├── order-service-dev.yml      # order-service en perfil dev
├── order-service-prod.yml     # order-service en perfil prod
├── user-service.yml
└── payment-service.yml
```

```yaml
# application.yml (config global)
logging:
  level:
    root: INFO

management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics
```

```yaml
# order-service.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/orders_db
    username: ${DB_USER}
    password: ${DB_PASSWORD}

app:
  orders:
    max-items: 50
    timeout-minutes: 30
```

## Config Client (cada microservicio)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

```yaml
# bootstrap.yml (carga antes que application.yml)
# O en application.yml con spring.config.import
spring:
  config:
    import: optional:configserver:http://localhost:8888
  application:
    name: order-service
  profiles:
    active: dev
  cloud:
    config:
      fail-fast: true   # fallar si no se puede conectar
      retry:
        max-attempts: 6
        initial-interval: 1000
        multiplier: 1.1
        max-interval: 2000
```

## Endpoint del Config Server

```bash
# Obtener config manualmente:
GET http://localhost:8888/{application}/{profile}/{label}

# Ejemplos:
GET http://localhost:8888/order-service/default
GET http://localhost:8888/order-service/dev
GET http://localhost:8888/order-service/prod/main

# Respuesta:
{
  "name": "order-service",
  "profiles": ["dev"],
  "propertySources": [
    { "name": "order-service-dev.yml", "source": {...} },
    { "name": "order-service.yml", "source": {...} },
    { "name": "application.yml", "source": {...} }
  ]
}
```

## Refresh dinámico sin reiniciar

```xml
<!-- En cada microservicio cliente -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-kafka</artifactId>
    <!-- O spring-cloud-starter-bus-amqp para RabbitMQ -->
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: refresh, busrefresh
```

```java
// Las beans que usan @ConfigurationProperties o @Value se refrescan
@RefreshScope  // ← necesario para que se refresque
@Service
public class OrderService {

    @Value("${app.orders.max-items}")
    private int maxItems;
}
```

```bash
# Refrescar un servicio específico:
POST http://localhost:8082/actuator/refresh

# Refrescar TODOS los servicios via Spring Cloud Bus:
POST http://localhost:8888/actuator/busrefresh
# → Config Server publica evento en Kafka/RabbitMQ
# → todos los servicios escuchan y se refrescan automáticamente
```

## Config Server con HashiCorp Vault (secrets)

```yaml
spring:
  cloud:
    config:
      server:
        vault:
          host: localhost
          port: 8200
          kvVersion: 2
          backend: secret
```

```bash
# Guardar secret en Vault:
vault kv put secret/order-service db.password=supersecret jwt.secret=myjwtsecret

# El Config Server lo expone al cliente como propiedad normal
```

## 📌 Notas Rápidas
- `@RefreshScope` es imprescindible para que un bean recargue su configuración con `/actuator/refresh` — sin esa anotación, el valor queda "congelado" desde el arranque aunque el Config Server tenga el dato actualizado.
- Spring Cloud Bus + `busrefresh` propaga el refresh a **todas** las instancias de todos los servicios con un solo request, usando Kafka/RabbitMQ como canal — evita tener que pegarle una por una a cada instancia.
- En un contexto Kubernetes/AKS, ConfigMaps y Secrets (ver [[04 - ConfigMaps Secrets y Volumenes]]) suelen cumplir buena parte de este rol de forma nativa — Config Server sigue aportando valor cuando se necesita refresh en caliente sin reiniciar Pods, algo que un ConfigMap montado como volumen no siempre logra de forma inmediata según cómo la app lee el archivo.
- Vault como backend de secrets (en vez de guardarlos en el repo Git de config) es la práctica correcta — un repo Git de configuración con secrets en texto plano es un riesgo de seguridad grave.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

