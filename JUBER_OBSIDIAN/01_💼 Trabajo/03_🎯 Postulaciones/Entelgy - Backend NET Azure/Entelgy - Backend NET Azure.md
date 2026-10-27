---
tags:
  - Postulacion
  - Entelgy
  - Backend
  - dotNET
  - Azure
  - SQLServer
  - Microservicios
Creado: 2026-08-30
Estado: Por postular
Empresa: Entelgy
Modalidad: No indicada
Ubicacion: No indicada
Sueldo: No indicado
Seniority: Semi Senior
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - NET
  - Azure
---

# Entelgy — Backend .NET Core Semi Senior

## Veredicto rápido

**Postular: sí, prioridad máxima.**

Es la coincidencia backend más directa con la ruta principal: ASP.NET Core, EF Core, Dapper, SQL Server, Azure, REST, microservicios, seguridad, GitHub Actions, observabilidad y arquitectura. No añade un lenguaje nuevo ni exige experiencia sectorial.

La vacante no indica años mínimos, sueldo, modalidad ni ubicación. Esos datos deben confirmarse, pero técnicamente debe estar entre las primeras postulaciones.

## Encaje estimado

| Escenario | Encaje aproximado |
|---|---:|
| Experiencia profesional .NET/SQL Server/Azure y APIs | 85-95 % |
| .NET/SQL Server sólidos; brechas puntuales en Azure o Dapper | 75-88 % |
| .NET sólido con Azure principalmente de laboratorio | 65-80 % |
| Sin experiencia backend profesional comprobable | Riesgo alto |

## Matriz de requisitos

| Capacidad | Importancia | Evidencia necesaria |
|---|---|---|
| C#/.NET Core | Crítica | APIs mantenibles y casos reales |
| ASP.NET Core Web API | Crítica | contratos, validación, errores y seguridad |
| EF Core + Dapper | Crítica | criterio para elegir y combinar |
| SQL Server | Crítica | SQL complejo, procedures, índices y tuning |
| Azure | Crítica | despliegue, secretos, mensajería y monitoreo |
| REST/microservicios | Alta | diseño, resiliencia y operación |
| JWT/OAuth2/Entra ID | Alta | autenticación y autorización aplicadas |
| Git/GitHub | Alta | branching, PRs y code review |
| GitHub Actions | Alta | pipeline ejecutable y despliegue |
| Clean Architecture/DDD/CQRS | Alta | aplicación con trade-offs, no definiciones |
| Observabilidad | Alta | App Insights, métricas, logs y trazas |

## C# y ASP.NET Core Web API

Preparar:

- C# moderno y programación asíncrona;
- DI, middleware y configuración;
- controllers o minimal APIs según contexto;
- DTOs y separación del dominio;
- validación y Problem Details;
- filtros, manejo global de errores y logging;
- versionado, paginación y OpenAPI;
- health checks y graceful shutdown;
- unit e integration tests;
- performance y cancelación.

Material: [[ASP.NET Core APIs]], [[Integration Tests WebApplicationFactory]], [[02 - HttpClientFactory Refit y gRPC]], [[03 - Resiliencia con Polly]].

## EF Core y Dapper

### EF Core

- modelado y configuración Fluent API;
- tracking frente a `AsNoTracking`;
- proyecciones;
- relaciones y carga de datos;
- migrations;
- transacciones y concurrencia optimista;
- evitar N+1;
- inspección de SQL generado;
- compiled queries solo después de medir.

### Dapper

- consultas parametrizadas;
- mapping de resultados;
- multi-mapping cuando sea necesario;
- transactions y connection lifecycle;
- invocación de stored procedures;
- consultas explícitas de alto rendimiento;
- evitar concatenación SQL e infraestructura duplicada.

### Criterio de elección

| Caso | Preferencia inicial |
|---|---|
| CRUD y modelo rico con unit of work | EF Core |
| Query compleja o lectura optimizada | Dapper |
| Procedure existente | Dapper u ODP específico según driver |
| Necesidad de ambos | patrón CQRS pragmático con ownership claro |

No presentar Dapper como automáticamente más rápido ni mezclar ambos sin una razón medible.

## SQL Server

- joins, CTE, subqueries y window functions;
- procedures y funciones;
- índices clustered/nonclustered e índices compuestos;
- planes de ejecución;
- estadísticas, cardinalidad y parameter sniffing;
- transacciones, aislamiento, locks y deadlocks;
- queries parametrizadas;
- paginación y lotes;
- connection pooling;
- diagnóstico con herramientas de SQL Server;
- migrations compatibles con despliegue.

Material: [[SQL Server con NET y EF Core]], [[01 - Conexion desde NET]], [[02 - EF Core Migrations]], [[03 - LINQ vs Stored Procedures]], [[04 - Proyecto API NET con EF Core]].

## Azure — ruta exacta para la vacante

### App Service

- despliegue de API;
- configuración por ambiente;
- managed identity;
- deployment slots;
- health checks;
- autoscaling;
- logs y rollback.

### Azure Functions

- triggers y bindings;
- funciones HTTP y Service Bus;
- idempotencia y retries;
- timeout y plan de hosting;
- observabilidad y poison messages.

### Key Vault

- secretos, certificados y llaves;
- acceso mediante managed identity;
- mínimo privilegio;
- rotación;
- no copiar secretos al pipeline o repositorio.

### Service Bus

- queues y topics/subscriptions;
- peek-lock y settlement;
- retries y dead-letter queue;
- duplicate detection e idempotencia;
- sessions si se requiere orden;
- métricas de cola y alertas.

### Azure Monitor/Application Insights

- logs estructurados;
- request/dependency telemetry;
- métricas y dashboards;
- distributed tracing;
- correlation ID;
- alertas accionables;
- diagnóstico de latencia y fallos.

Material: [[06 - Azure para un Full Stack empleable]], [[08 - Observabilidad logs métricas y trazas]], [[Mensajeria RabbitMQ Azure Service Bus Kafka]].

## Autenticación y autorización

Debes distinguir:

- OAuth 2.0 como framework de autorización;
- OpenID Connect para identidad;
- JWT como formato de token;
- Microsoft Entra ID como proveedor de identidad;
- app registration, scopes, roles y consent;
- delegated frente a application permissions;
- authorization code + PKCE para usuario;
- client credentials para servicio a servicio;
- validación de issuer, audience, firma y expiración;
- autorización por policies/claims en la API;
- managed identity para recursos Azure.

Nunca limitar la seguridad a decodificar un JWT o esconder endpoints en frontend.

## Git, GitHub y GitHub Actions

### Flujo colaborativo

- ramas pequeñas;
- commits claros;
- pull requests revisables;
- branch protection;
- required checks;
- code review;
- resolución de conflictos;
- tags/releases.

### Pipeline mínimo

```text
pull request
  -> restore
  -> build
  -> unit tests
  -> integration tests
  -> lint/static analysis
  -> security/dependency scan
  -> artifact o container
  -> deploy por ambiente
  -> smoke test
  -> monitoreo y rollback
```

Gestionar autenticación hacia Azure mediante identidad federada/OIDC cuando el entorno lo permita, evitando secretos de larga duración.

## Clean Architecture, DDD y CQRS

### Clean Architecture

- dependencias hacia el núcleo;
- dominio y casos de uso sin infraestructura;
- adapters externos;
- pruebas por límite;
- evitar proyectos/capas sin responsabilidad real.

### DDD

- lenguaje ubicuo;
- bounded contexts;
- agregados e invariantes;
- entidades/value objects;
- domain events;
- repositorios como puertos;
- anticorruption layer para integraciones.

### CQRS

- separar modelos de lectura/escritura solo cuando aporta claridad;
- comandos que expresan intención;
- queries sin efectos laterales;
- handlers delgados y dominio responsable;
- Dapper para lecturas y EF Core para escritura como opción, no regla;
- no implica obligatoriamente dos bases, eventos ni event sourcing.

Material: [[10 - Arquitectura y sistemas distribuidos]].

## Microservicios

- límites por negocio;
- datos y ownership;
- contratos compatibles;
- comunicación síncrona/asíncrona;
- timeout, retry, circuit breaker e idempotencia;
- consistencia eventual y Outbox;
- seguridad service-to-service;
- trazabilidad distribuida;
- despliegue y rollback independiente;
- cuándo un monolito modular es suficiente.

## Proyecto demostrable

Construir o adaptar una API de órdenes:

- ASP.NET Core Web API;
- dominio y casos de uso separados;
- EF Core para escritura;
- Dapper para dashboard de lectura;
- SQL Server con índices y procedure justificado;
- Entra ID/JWT y policies;
- evento hacia Azure Service Bus;
- Azure Function consumidora e idempotente;
- Key Vault + managed identity;
- App Service;
- App Insights con trazas;
- GitHub Actions con tests y despliegue;
- ADR sobre Clean Architecture, CQRS y límites de servicio.

Este único proyecto cubre casi todo el anuncio.

## Preparación de siete días

| Día | Foco | Entregable |
|---|---|---|
| 1 | CV y API | historias .NET/SQL/Azure y API base |
| 2 | EF Core + Dapper | escritura EF y lectura Dapper medidas |
| 3 | SQL Server | procedure, índices y plan de ejecución |
| 4 | Seguridad | Entra ID, scopes/roles y policies |
| 5 | Azure | App Service, Key Vault, Service Bus y Function |
| 6 | CI/CD/observabilidad | GitHub Actions y App Insights |
| 7 | Arquitectura | DDD/CQRS, demo y simulación técnica |

Postular inmediatamente.

## Preguntas probables

- ¿Cuándo elegirías Dapper frente a EF Core?
- ¿Cómo detectas y corriges N+1?
- ¿Cómo optimizas una consulta SQL Server?
- ¿Cómo diseñas un endpoint idempotente?
- ¿Cuál es la diferencia entre OAuth2, OIDC y JWT?
- ¿Cómo proteges una API con Entra ID?
- ¿Cómo accede App Service a Key Vault sin secretos?
- ¿Cómo procesas mensajes de Service Bus sin duplicar efectos?
- ¿Qué observas en Application Insights durante un incidente?
- ¿Cómo implementarías CQRS sin sobreingeniería?
- ¿Cuándo no usar microservicios?
- ¿Qué incluirías en GitHub Actions y cómo harías rollback?

## Palabras clave para el CV

`C#`, `.NET Core`, `ASP.NET Core Web API`, `Entity Framework Core`, `Dapper`, `SQL Server`, `stored procedures`, `query optimization`, `REST APIs`, `microservices`, `Azure App Service`, `Azure Functions`, `Key Vault`, `Service Bus`, `Azure Monitor`, `Application Insights`, `Entra ID`, `OAuth 2.0`, `JWT`, `GitHub Actions`, `Clean Architecture`, `DDD`, `CQRS`.

## Preguntas para reclutamiento

1. ¿Cuántos años de experiencia buscan para Semi Senior?
2. ¿Cuál es el rango salarial y tipo de contrato?
3. ¿Cuál es la modalidad, ubicación y frecuencia presencial?
4. ¿Qué versiones de .NET y SQL Server utilizan?
5. ¿Qué servicios Azure son de uso diario y cuáles deseables?
6. ¿Utilizan EF Core y Dapper dentro del mismo servicio?
7. ¿Los microservicios ya existen o se está migrando una solución?
8. ¿Quién administra Azure y los pipelines?
9. ¿Existe on-call o soporte fuera de horario?
10. ¿Qué nivel de DDD/CQRS está aplicado actualmente?

## Decisión final

Es la vacante de **mayor prioridad técnica** hasta ahora para la vía .NET backend. Coincide casi punto por punto con la ruta y convierte Azure, SQL Server, seguridad, observabilidad y CI/CD en evidencia integrada.

La acción correcta es adaptar el CV y postular sin esperar a completar nuevos estudios.
