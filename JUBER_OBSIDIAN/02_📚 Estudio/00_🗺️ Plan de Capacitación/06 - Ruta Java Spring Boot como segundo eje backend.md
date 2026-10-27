---
tags:
  - Postulaciones
  - Java
  - Spring
  - Entrevistas
Creado: 2026-08-30
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - 16 - Radar acumulativo de vacantes
---

# Ruta Java Spring Boot como segundo eje backend

## Por qué existe esta nota

El radar de vacantes ([[16 - Radar acumulativo de vacantes]]) confirma Java/Spring en **7 de 14** vacantes activas, contra 8 de .NET — prácticamente empatado. Vacantes de alta prioridad (Stefanini, TSOFT, Entelgy no lo pide pero Full Stack Senior Java, ISP Consulting, Java Senior BIAN, Backend Java NET DDD DB2) lo tratan como **eje principal**, no como alternativa de respaldo. Esta ruta no reconstruye Java desde cero: apunta al contenido ya preparado en `02_📚 Estudio/03_☕ Java/`, con el mismo criterio de "listo para entrevista" que usa [[04 - Ruta de repaso NET para entrevistas]].

> [!warning] Ajuste de expectativa — 31/08/2026
> El CV **no tiene experiencia profesional en Java backend** (ver [[23 - Mi CV frente a las 14 Vacantes]]). Esta ruta construye conocimiento real, pero **no convierte el CV en un perfil Java senior en 5 días**.
>
> Úsala para: vacantes **políglotas** donde .NET es la mitad del requisito, y para responder con solvencia si Java aparece en una entrevista. **No** para afirmar experiencia Java que no existe — se cae en la primera repregunta.

## Objetivo

Poder diseñar, implementar, probar y diagnosticar una API Spring Boot conectada a una base relacional, y explicar en entrevista las equivalencias con .NET — que es tu eje demostrable — cuando lo pidan.

## Día 1 — Java moderno y Spring Core

- Generics, Streams, Optional, records, `var`, concurrencia básica.
- IoC Container, Dependency Injection (constructor vs setter vs field), Beans y Scopes.
- AOP y el problema de auto-invocación.
- Lombok: qué genera en compile-time y dónde es riesgoso con entidades JPA.

Repasar: notas de `07_ Spring/SPRING-CORE/` (IoC Container, DI, Beans/Scopes, AOP, Lombok).

**Prueba:** explicar por qué `@Data` en una entidad JPA con relaciones puede romper `equals`/`hashCode` y causar bucles infinitos de serialización.

## Día 2 — Spring Boot y REST

- Fundamentos Spring Boot, `@RestController`, Request/Response, validaciones (`@Valid`, Bean Validation).
- Configuración/Properties, Profiles, Actuator, manejo de errores (`@ControllerAdvice`).
- Swagger/OpenAPI con `springdoc-openapi`.

Repasar: notas 01-09 de `07_ Spring/SPRING-BOOT/`.

**Prueba:** endpoint con validación, manejo de errores consistente y documentado en Swagger — el equivalente exacto al `ProblemDetails`/OpenAPI de .NET.

## Día 3 — Spring Data JPA y bases de datos

- Entity states, N+1, `FetchType`, relaciones.
- Repositorios (derived queries, JPQL, Specifications).
- Transacciones: Propagation, Isolation, reglas de rollback.
- Migraciones con Flyway/Liquibase.
- NoSQL: ACID vs BASE, CAP theorem, cuándo un documento gana a una tabla.

Repasar: notas de `07_ Spring/SPRING-DATA/` y `08_ Bases de Datos/05 - NoSQL desde Java`.

**Prueba:** detectar y corregir un N+1 en una relación `@OneToMany`, igual que el ejercicio de EF Core en la ruta .NET.

## Día 4 — Seguridad y resiliencia

- `SecurityFilterChain`, `UserDetailsService`, filtro JWT.
- OAuth2/OIDC, method security (`@PreAuthorize`), CORS/CSRF.
- Circuit Breaker con Resilience4j (equivalente Java de Polly).
- OWASP Top 10 mapeado a Java/Spring — ver [[03 - Seguridad Backend Java]].

Repasar: notas de `07_ Spring/SPRING-SECURITY/` y `10_ Microservicios/05 - Circuit Breaker - Resilience4j`.

**Prueba:** endpoint protegido por rol y por ownership del recurso (A01 Broken Access Control), con cliente externo envuelto en circuit breaker.

## Día 5 — Testing, mensajería y microservicios

- JUnit 5, Mockito, TDD, Testing en Spring Boot, TestContainers.
- Kafka y RabbitMQ con Spring Boot: partitions/consumer groups, exchanges, Dead Letter Topics/Exchanges, Outbox Pattern.
- Spring Cloud (Gateway, Eureka, Config Server, OpenFeign), Distributed Tracing.

Repasar: notas de `07_ Spring/TESTING/`, `09_ Mensajeria/` y `10_ Microservicios/`.

**Prueba:** integration test con TestContainers que cruza endpoint, autenticación y persistencia — el equivalente Java de `WebApplicationFactory`.

## Equivalencias rápidas .NET ↔ Java (para no mezclar vocabulario en entrevista)

| .NET | Java/Spring |
|---|---|
| ASP.NET Core Controllers/Minimal API | `@RestController` |
| EF Core | Spring Data JPA / Hibernate |
| `DbContext` (scoped) | `EntityManager` (request-scoped vía JPA) |
| `ProblemDetails` | `@ControllerAdvice` + `ExceptionHandler` |
| Polly (retry/circuit breaker) | Resilience4j |
| `IOptions<T>` | `@ConfigurationProperties` |
| Middleware pipeline | Filter chain / Interceptors |
| xUnit + `WebApplicationFactory` | JUnit 5 + `@SpringBootTest` |
| Azure Service Bus | Spring Kafka / Spring AMQP (RabbitMQ) |
| Dapper | JdbcTemplate / MyBatis |

Tener esta tabla clara evita el error más común en entrevista políglota: responder con el término .NET cuando preguntan por Java, o viceversa.

## Preguntas que debes defender

- ¿Por qué `@Data` es riesgoso en una entidad JPA con relaciones bidireccionales?
- ¿Qué diferencia hay entre `@Autowired` por constructor y por campo, y por qué se prefiere constructor?
- ¿Cómo evitas un N+1 en una relación `@OneToMany`?
- ¿Cómo diseñarías autorización por recurso, no solo por rol, en Spring Security?
- ¿Cómo garantizas que un consumer de Kafka no procese el mismo mensaje dos veces?
- ¿Cuándo elegirías Quarkus sobre Spring Boot? (ver [[Quarkus]] si la vacante lo menciona explícitamente)

Continuar con el banco de preguntas backend de [[15 - Banco de preguntas Full Stack]].

## Criterio de salida

- [ ] Construyo una API Spring Boot segura y consistente sin tutorial, al mismo nivel que en .NET.
- [ ] Puedo explicar el SQL/HQL generado por Hibernate y corregir un N+1.
- [ ] Tengo pruebas de integración reales con TestContainers.
- [ ] Puedo mapear cualquier término .NET a su equivalente Java sin dudar, y explicar el porqué de la diferencia, no solo el nombre.
- [ ] Puedo sostener 45 segundos de pitch "Backend Java/Spring" sin sonar traducido del pitch de .NET.
