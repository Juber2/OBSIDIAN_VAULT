---
tags:
  - Postulaciones
  - dotNET
  - Entrevistas
Creado: 2026-08-30
Relacionado:
  - NET
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Ruta de repaso .NET para entrevistas

## Objetivo

Poder diseñar, implementar, probar y diagnosticar una API ASP.NET Core conectada a una base relacional.

## Día 1 — C# y runtime

- Tipos por valor/referencia, records y nullability.
- Generics, colecciones y LINQ.
- `async`/`await`, `Task`, `CancellationToken` y excepciones.
- GC y allocations a nivel conceptual.

Repasar: [[CSharp y Runtime dotNET]], [[LINQ colecciones y consultas]], [[Async Await Task CancellationToken]].

**Prueba:** implementar una consulta asíncrona cancelable, tipada y sin bloquear el hilo.

## Día 2 — ASP.NET Core

- Hosting, middleware y orden del pipeline.
- DI, lifetimes y Options Pattern.
- Controllers vs Minimal APIs.
- Routing, model binding, validación, errores y OpenAPI.

Repasar: [[Pipeline Middleware Hosting]], [[Dependency Injection Options Validation]], [[ProblemDetails errores y OpenAPI]].

**Prueba:** endpoint con validación, error consistente y documentación.

## Día 3 — EF Core y SQL

- `DbContext`, tracking/no-tracking y proyecciones.
- Relaciones, N+1, índices y consultas generadas.
- Migrations, transacciones, concurrencia e idempotencia.

Repasar: [[DbContext entidades y configuracion]], [[Queries tracking includes y proyecciones]], [[Transacciones concurrencia e idempotencia]].

**Prueba:** detectar y corregir una consulta N+1 o una lectura que carga columnas innecesarias.

## Día 4 — Seguridad y resiliencia

- Authentication vs authorization.
- JWT, refresh rotation, OAuth 2.0/OIDC y policies.
- `HttpClientFactory`, timeouts, retry, circuit breaker y límites.

Repasar: [[JWT Bearer Access Token]], [[Policies RBAC permisos y recursos]], [[05 - OAuth2 OpenID Connect]], [[03 - Resiliencia con Polly]].

**Prueba:** endpoint autorizado por policy y cliente externo con timeout explícito.

## Día 5 — Testing, performance y observabilidad

- xUnit y diseño de pruebas.
- Integration tests con `WebApplicationFactory`.
- Testcontainers y base real.
- Logs estructurados, health checks, métricas y trazas.

Repasar: [[Unit Testing xUnit FluentAssertions]], [[Integration Tests WebApplicationFactory]], [[Testcontainers y base de datos real]], [[Logging tracing metrics OpenTelemetry]].

**Prueba:** integration test que cruza endpoint, autenticación y persistencia.

## Preguntas que debes defender

- ¿Qué lifetime usarías para `DbContext` y por qué?
- ¿Qué ocurre internamente con una operación async?
- ¿Cuándo usar `AsNoTracking`?
- ¿Cómo manejas errores sin devolver detalles sensibles?
- ¿Cómo evitas duplicar una operación por reintentos?
- ¿Qué probarías con mocks y qué con integración?

Continuar con [[Preguntas de Entrevista dotNET]] y [[Preguntas Arquitectura Senior]].

## Criterio de salida

- [ ] Construyo una API segura y consistente sin tutorial.
- [ ] Puedo explicar el SQL generado por EF Core.
- [ ] Tengo pruebas de integración reales.
- [ ] Diagnostico un error usando logs y trazas.
