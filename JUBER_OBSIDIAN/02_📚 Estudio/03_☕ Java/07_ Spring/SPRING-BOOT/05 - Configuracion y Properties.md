---
tags:
  - Java
  - Spring Boot
Creado: 2026-05-21
Relacionado:
  - Spring Boot
  - Profiles
Palabras clave:
  - application.properties, application.yml, @ConfigurationProperties, @Value, Environment
---

## 🧠 application.properties vs application.yml

```properties
# application.properties
server.port=8080
spring.datasource.url=jdbc:postgresql://localhost:5432/midb
spring.datasource.username=postgres
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
```

```yaml
# application.yml (equivalente, más legible para configuraciones anidadas)
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/midb
    username: postgres
    password: secret
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
```

## @ConfigurationProperties — propiedades tipadas (recomendado)

```yaml
# application.yml
app:
  seguridad:
    jwt-secret: mi-secreto-muy-largo
    expiracion-horas: 24
  correo:
    host: smtp.gmail.com
    puerto: 587
    remitente: no-reply@miapp.com
  max-intentos-login: 5
```

```java
@ConfigurationProperties(prefix = "app")
@Component
// o usar @EnableConfigurationProperties(AppProperties.class) en @SpringBootApplication
public class AppProperties {

    private final Seguridad seguridad = new Seguridad();
    private final Correo correo = new Correo();
    private int maxIntentosLogin;

    @Getter @Setter
    public static class Seguridad {
        private String jwtSecret;
        private int expiracionHoras;
    }

    @Getter @Setter
    public static class Correo {
        private String host;
        private int puerto;
        private String remitente;
    }
}

// Uso
@Service
public class JwtService {
    @Autowired
    private AppProperties props;

    public String generarToken(String usuario) {
        return Jwts.builder()
            .setExpiration(Date.from(Instant.now().plusSeconds(
                props.getSeguridad().getExpiracionHoras() * 3600L)))
            .signWith(SignatureAlgorithm.HS256, props.getSeguridad().getJwtSecret())
            .compact();
    }
}
```

## @Value — valores individuales

```java
@Service
public class ConfigService {
    @Value("${app.nombre:Mi App}")          // con valor por defecto
    private String appNombre;

    @Value("${server.port}")
    private int puerto;

    @Value("${app.lista:valor1,valor2}")
    private List<String> lista;

    @Value("#{systemProperties['user.home']}") // SpEL
    private String homeDir;
}
```

## Variables de entorno

Spring Boot carga automáticamente variables de entorno. `APP_DATASOURCE_URL` mapea a `app.datasource.url`.

```bash
# En Docker o servidor
export SPRING_DATASOURCE_URL=jdbc:postgresql://prod-db:5432/midb
export SPRING_DATASOURCE_PASSWORD=secreto-produccion
java -jar mi-app.jar
```

## Jerarquía de prioridad (mayor a menor)

```
1. Command line args (--server.port=9090)
2. Variables de entorno (SERVER_PORT=9090)
3. application-{profile}.properties
4. application.properties
5. Valores por defecto en @Value
```

## Propiedades importantes de Spring Boot

```properties
# Servidor
server.port=8080
server.servlet.context-path=/api
server.compression.enabled=true

# JPA
spring.jpa.hibernate.ddl-auto=validate  # never/validate/update/create/create-drop
spring.jpa.show-sql=true
spring.jpa.open-in-view=false  # IMPORTANTE: desactivar en producción

# Logging
logging.level.root=INFO
logging.level.com.juber=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.file.name=logs/app.log

# Jackson
spring.jackson.serialization.indent-output=true
spring.jackson.default-property-inclusion=non_null

# Actuator
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=always
```

## 📌 Notas Rápidas
