---
tags:
  - Java
  - Quarkus
Creado: 2026-08-09
Relacionado:
  - Quarkus
---

# Configuración y Profiles

Quarkus usa **MicroProfile Config** (estándar), configurado principalmente en `application.properties` (o `.yaml` si se agrega la extension de YAML) — no `application.yml` por default como Spring Boot.

## application.properties básico

```properties
quarkus.http.port=8080
quarkus.application.name=mi-servicio

# Datasource
quarkus.datasource.db-kind=postgresql
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/midb

# Logging
quarkus.log.level=INFO
quarkus.log.category."com.ejemplo".level=DEBUG
```

## Inyectar configuración en código

```java
@ApplicationScoped
public class MailService {

    @ConfigProperty(name = "mail.remitente")
    String remitente;

    @ConfigProperty(name = "mail.timeout", defaultValue = "5000")
    int timeout;
}
```

Equivalente a `@Value` de Spring, pero con `@ConfigProperty` (estándar MicroProfile).

## Config record (tipado, agrupado)

```java
@ConfigMapping(prefix = "mail")
public interface MailConfig {
    String remitente();
    int timeout();
    Optional<String> replyTo();
}
```

```java
@Inject
MailConfig mailConfig;
```

Más seguro que inyectar propiedades sueltas — errores de nombre de propiedad se detectan en build time, no en runtime.

## Profiles (equivalente a Spring profiles)

Quarkus trae 3 profiles built-in: `dev`, `test`, `prod`.

```properties
# application.properties (aplica a todos)
quarkus.http.port=8080

# Solo en dev
%dev.quarkus.log.level=DEBUG
%dev.quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/midb_dev

# Solo en prod
%prod.quarkus.datasource.jdbc.url=${DATABASE_URL}
%prod.quarkus.log.level=WARN
```

El prefijo `%profile.` reemplaza el `application-{profile}.properties` separado de Spring — todo puede vivir en un solo archivo, aunque también se puede separar en archivos por profile si se prefiere.

```bash
./mvnw quarkus:dev              # corre en profile "dev" automáticamente
java -Dquarkus.profile=prod -jar mi-servicio.jar
```

## Variables de entorno y overrides

MicroProfile Config tiene un orden de precedencia claro (de mayor a menor prioridad):
1. Variables de entorno del sistema (`DATABASE_URL`)
2. System properties (`-Dquarkus.datasource.jdbc.url=...`)
3. `application.properties`
4. Defaults en el código

```bash
export QUARKUS_DATASOURCE_JDBC_URL=jdbc:postgresql://prod-host:5432/db
```

Las variables de entorno se mapean automáticamente (mayúsculas, `_` en vez de `.` y `-`).

## Secrets — no hardcodear en application.properties

Para producción, las credenciales no van en el archivo de properties commiteado — se inyectan vía variables de entorno (desde Kubernetes Secrets, Vault, etc.), siguiendo el mismo principio que en cualquier stack: **configuración externalizada del código**.

## Errores comunes

- Hardcodear URLs/credenciales de producción en `application.properties` en vez de usar `%prod.` + variables de entorno.
- No usar `@ConfigMapping` para grupos de configuración relacionados, terminando con `@ConfigProperty` sueltos por todos lados sin cohesión.
- Confundir el prefijo `%profile.` (Quarkus) con el sufijo `-profile` de archivos separados (Spring) al buscar documentación o ejemplos.

## Checklist
- [ ] Sé configurar `application.properties` con profiles (`%dev.`, `%prod.`).
- [ ] Uso `@ConfigProperty` y `@ConfigMapping` según corresponda.
- [ ] Entiendo el orden de precedencia de MicroProfile Config.
- [ ] No tengo secretos de producción hardcodeados en el repo.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

