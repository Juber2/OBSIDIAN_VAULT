---
tags:
  - Postulaciones
  - Entrevistas
  - FullStack
Creado: 2026-08-30
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Banco de preguntas Full Stack

Responder en voz alta con definición, ejemplo propio y trade-off.

## Angular y TypeScript

1. ¿Cuándo usar Signals, RxJS o NgRx?
2. ¿Qué hace `switchMap` y cuándo sería incorrecto?
3. ¿Cómo cancelas solicitudes obsoletas?
4. ¿Cómo estructuras una feature grande?
5. ¿Qué diferencia hay entre un guard y autorización?
6. ¿Cómo implementas un interceptor de refresh sin bucle infinito?
7. ¿Qué ventaja ofrecen typed reactive forms?
8. ¿Cómo diagnosticas renders innecesarios?
9. ¿Qué estrategias de lazy loading y rendering conoces?
10. ¿Qué probarías en un componente y qué dejarías para E2E?

## C# y ASP.NET Core

1. ¿Qué ocurre con `async`/`await` y por qué evitar `.Result`?
2. ¿Cómo se propaga `CancellationToken`?
3. ¿Qué diferencias hay entre transient, scoped y singleton?
4. ¿Por qué `DbContext` suele ser scoped?
5. ¿Controller o Minimal API?
6. ¿Cómo implementas errores consistentes con `ProblemDetails`?
7. ¿Authentication y authorization son lo mismo?
8. ¿Cómo diseñas una policy por recurso?
9. ¿Qué causa un N+1 en EF Core?
10. ¿Cuándo usar `AsNoTracking` y proyecciones?
11. ¿Cómo haces idempotente un endpoint?
12. ¿Qué probarías con `WebApplicationFactory`?

## Java y Spring Boot

1. ¿Por qué preferís inyección por constructor en vez de `@Autowired` por campo?
2. ¿Qué problema causa `@Data` de Lombok en una entidad JPA con relaciones?
3. ¿Qué diferencia hay entre `@Transactional` con propagación `REQUIRED` y `REQUIRES_NEW`?
4. ¿Qué causa un N+1 en una relación `@OneToMany` y cómo lo corregís?
5. ¿Cómo diseñarías autorización por recurso con `@PreAuthorize`, no solo por rol?
6. ¿Cuándo usarías Resilience4j y qué patrón resuelve cada anotación?
7. ¿Qué garantiza que un consumer de Kafka no procese dos veces el mismo mensaje?
8. ¿Qué diferencia hay entre `@Component`, `@Service` y `@Repository` más allá de la semántica?
9. ¿Spring Boot o Quarkus para un proyecto Cloud Native con arranque rápido?
10. ¿Qué probarías con `@SpringBootTest` y qué con un test unitario puro?
11. ¿Cómo mapearías un término .NET (por ejemplo `IOptions<T>`) a su equivalente Java?

## SQL y datos

1. ¿Qué índice crearías y cómo verificarías que se utiliza?
2. ¿Qué diferencias existen entre optimistic y pessimistic concurrency?
3. ¿Qué es un deadlock y cómo se reduce?
4. ¿Qué nivel de aislamiento elegirías?
5. ¿SQL Server o PostgreSQL para este caso?
6. ¿Cuándo conviene un documento en lugar de un modelo relacional?
7. ¿Por qué MongoDB no es automáticamente la solución para logs?
8. ¿Cómo ejecutas una migración zero-downtime?
9. ¿Qué problema resuelve connection pooling?
10. ¿Cómo investigarías una query lenta?

## Azure, Docker y DevOps

1. ¿App Service, Container Apps o AKS?
2. ¿Cómo usa una app managed identity?
3. ¿Qué secretos pertenecen en Key Vault?
4. ¿Cómo organizarías build, test y deploy?
5. ¿Qué es un Docker multi-stage build?
6. ¿Liveness y readiness son iguales?
7. ¿Cómo harías rollback?
8. ¿Bicep o Terraform?
9. ¿Cuándo usarías Service Bus?
10. ¿Cómo evitarías procesar dos veces un mensaje?

## Observabilidad y producción

1. ¿Qué diferencia logs, métricas y trazas?
2. ¿Cómo correlacionas una solicitud entre servicios?
3. ¿Qué métricas mostrarías en un dashboard?
4. ¿Qué convierte una alerta en accionable?
5. ¿Qué es cardinalidad y por qué importa?
6. ¿Qué información jamás registrarías?
7. ¿Cómo investigarías un aumento de latencia p95?
8. ¿Qué son SLI y SLO?
9. ¿Cómo controlas costo y volumen de telemetría?
10. Describe un incidente desde alerta hasta postmortem.

## Arquitectura

1. ¿Modular monolith o microservicios?
2. ¿Cuándo aplicarías Cache-Aside?
3. ¿Qué problema resuelve Outbox?
4. ¿Retry siempre mejora resiliencia?
5. ¿Qué diferencia timeout, retry y circuit breaker?
6. ¿Cuándo aceptar consistencia eventual?
7. ¿Qué responsabilidad tiene un API Gateway?
8. ¿Cómo aislarías datos multitenant?
9. ¿Cómo evolucionas un contrato sin romper clientes?
10. Diseña una plataforma full stack observable y segura.

## Conductuales

1. Cuéntame un error técnico que cometiste.
2. Describe un desacuerdo de arquitectura.
3. ¿Cómo priorizas deuda técnica frente a una entrega?
4. Cuéntame un incidente difícil.
5. ¿Cómo haces code review sin bloquear al equipo?
6. ¿Qué aprendiste recientemente y cómo lo aplicaste?
7. ¿Cómo comunicas un riesgo a una persona no técnica?
8. ¿Qué harías durante tus primeros 30 días?

