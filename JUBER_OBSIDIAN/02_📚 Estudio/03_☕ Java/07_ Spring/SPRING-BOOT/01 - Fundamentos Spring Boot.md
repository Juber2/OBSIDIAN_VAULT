---
tags:
  - Java
  - Spring Boot
Creado: 2026-05-21
Relacionado:
  - Spring Boot
Palabras clave:
  - Spring Boot, auto-configuration, starter, @SpringBootApplication, fat JAR
---

## 🧠 Qué es Spring Boot

Spring Boot elimina la configuración manual de Spring. Provee:
- **Auto-configuration**: detecta las librerías en el classpath y configura automáticamente
- **Starters**: dependencias agrupadas por funcionalidad
- **Embedded server**: Tomcat, Jetty o Undertow embebido
- **Actuator**: métricas y monitoreo

## @SpringBootApplication

```java
@SpringBootApplication
// es equivalente a:
// @Configuration
// @EnableAutoConfiguration
// @ComponentScan
public class MiApp {
    public static void main(String[] args) {
        SpringApplication.run(MiApp.class, args);
    }
}
```

## Starters principales

| Starter | Para qué |
|---------|----------|
| `spring-boot-starter-web` | REST APIs, MVC (incluye Tomcat) |
| `spring-boot-starter-data-jpa` | JPA, Hibernate |
| `spring-boot-starter-security` | Autenticación y autorización |
| `spring-boot-starter-test` | JUnit 5, Mockito, AssertJ |
| `spring-boot-starter-validation` | Bean Validation |
| `spring-boot-starter-actuator` | Métricas y health checks |
| `spring-boot-starter-cache` | Caché abstracta |
| `spring-boot-starter-mail` | Envío de emails |
| `spring-boot-starter-webflux` | Programación reactiva |

## Auto-configuration

Spring Boot tiene >100 `@AutoConfiguration` classes. Ejemplo de cómo funciona:

```java
// Spring Boot detecta que tienes H2 + JPA → configura DataSource automáticamente
// Con esta config en application.properties:
spring.datasource.url=jdbc:h2:mem:testdb
// Spring Boot crea el DataSource, SessionFactory, TransactionManager, etc.
```

```java
// Ver qué auto-configuraciones aplican:
./mvnw spring-boot:run --debug 2>&1 | grep "Auto-configuration"
```

## Ciclo de inicio

```
main() → SpringApplication.run()
        → Carga Environment (properties, env vars)
        → Crea ApplicationContext
        → Ejecuta AutoConfiguration
        → Escanea @Components
        → Levanta servidor embebido (si es web)
        → ApplicationReadyEvent
```

## Personalizar el inicio

```java
@SpringBootApplication
public class MiApp {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MiApp.class);
        app.setBannerMode(Banner.Mode.OFF); // sin banner
        app.setDefaultProperties(Map.of("server.port", "9090"));
        app.run(args);
    }
}

// O con builder
@SpringBootApplication
public class MiApp {
    public static void main(String[] args) {
        new SpringApplicationBuilder(MiApp.class)
            .bannerMode(Banner.Mode.CONSOLE)
            .logStartupInfo(true)
            .run(args);
    }
}
```

## ApplicationRunner / CommandLineRunner

```java
// Ejecutar código después de que la app arranca
@Component
public class DataInitializer implements ApplicationRunner {
    @Autowired
    private UsuarioRepository repository;

    @Override
    public void run(ApplicationArguments args) {
        if (repository.count() == 0) {
            repository.save(new Usuario("admin", "admin@email.com"));
        }
    }
}
```

## Fat JAR

Spring Boot empaqueta todo en un único JAR ejecutable:

```bash
mvn clean package
java -jar target/mi-app-1.0.0.jar

# Con propiedades externas
java -jar mi-app.jar --spring.profiles.active=prod
java -jar mi-app.jar --server.port=9090
```

## 📌 Notas Rápidas
