---
tags:
  - Java
  - Spring Boot
Creado: 2026-05-21
Relacionado:
  - Spring Boot
  - Configuracion y Properties
Palabras clave:
  - profiles, @Profile, spring.profiles.active, dev, prod, test
---

## 🧠 Concepto

Los **perfiles** permiten tener configuraciones diferentes para distintos entornos (dev, test, prod) sin cambiar el código.

## Archivos de configuración por perfil

```
src/main/resources/
├── application.properties          → configuración base (siempre se carga)
├── application-dev.properties      → solo cuando perfil = dev
├── application-test.properties     → solo cuando perfil = test
└── application-prod.properties     → solo cuando perfil = prod
```

```properties
# application.properties (base)
spring.application.name=mi-app
app.nombre=Mi Aplicación

# application-dev.properties
server.port=8080
spring.datasource.url=jdbc:h2:mem:devdb
spring.jpa.show-sql=true
logging.level.com.juber=DEBUG

# application-prod.properties
server.port=80
spring.datasource.url=jdbc:postgresql://prod-db:5432/miapp
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
spring.jpa.show-sql=false
logging.level.root=WARN
```

## Activar el perfil

```bash
# Al ejecutar
java -jar mi-app.jar --spring.profiles.active=prod

# Variable de entorno
export SPRING_PROFILES_ACTIVE=prod

# En application.properties (no recomendado para prod)
spring.profiles.active=dev

# En tests
@ActiveProfiles("test")
class MiTest { }
```

## @Profile — beans condicionales por perfil

```java
// Solo activo en desarrollo
@Component
@Profile("dev")
public class MockEmailService implements EmailService {
    public void enviar(String destino, String asunto, String cuerpo) {
        System.out.println("[MOCK] Email a: " + destino + " | Asunto: " + asunto);
    }
}

// Solo activo en producción
@Component
@Profile("prod")
public class SmtpEmailService implements EmailService {
    // implementación real con SMTP
}

// En múltiples perfiles
@Component
@Profile({"dev", "test"})
public class DataSeeder { ... }

// Negación — todos menos producción
@Component
@Profile("!prod")
public class DebugController { ... }
```

## Perfiles en @Configuration

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }

    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl(System.getenv("DB_URL"));
        ds.setUsername(System.getenv("DB_USER"));
        ds.setPassword(System.getenv("DB_PASSWORD"));
        ds.setMaximumPoolSize(20);
        return ds;
    }
}
```

## Múltiples perfiles activos

```bash
java -jar app.jar --spring.profiles.active=prod,monitoring,feature-x
```

## 📌 Notas Rápidas
