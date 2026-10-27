---
tags:
  - Postulaciones
  - Roadmap
  - FullStack
Creado: 2026-08-30
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Prioridades del stack

## P0 — Núcleo que decide la contratación

### Frontend

- JavaScript, TypeScript y asincronía.
- Angular moderno: standalone components, templates, DI y routing.
- Reactive Forms, HttpClient, interceptors y manejo de errores.
- RxJS y Signals con límites claros.
- Estado de loading, error, empty y permisos.
- Accesibilidad, responsive y fundamentos de performance.
- Pruebas de componentes, servicios y flujos críticos.

### Backend

- C#, LINQ, async/await y `CancellationToken`.
- ASP.NET Core: middleware, DI, configuración y APIs REST.
- Validación, `ProblemDetails`, OpenAPI, paginación y versionado.
- EF Core, SQL, índices, transacciones y concurrencia.
- JWT/OIDC, policies, roles y seguridad de APIs.
- Unit tests e integration tests.

### Entrega

- Git y pull requests.
- Dockerfile multi-stage y Docker Compose.
- Pipeline build → test → image → deploy.
- Azure App Service o Container Apps.
- Azure SQL, Key Vault, Storage y Application Insights.

## P1 — Diferenciadores con retorno alto

- PostgreSQL a nivel desarrollador y optimización básica.
- OpenTelemetry: logs, métricas, trazas y correlación.
- Redis y Cache-Aside.
- Azure Service Bus, background workers, reintentos e idempotencia.
- GitHub Actions o Azure DevOps.
- Bicep o Terraform básico.
- Modular Monolith, Clean Architecture pragmática y Outbox.
- Diagnóstico de performance y de incidentes.

## P2 — Aprender por demanda

- MongoDB/Azure DocumentDB/Cosmos DB para datos documentales.
- Kubernetes y AKS.
- API Management y redes avanzadas.
- Kafka.
- Microfrontends y microservicios complejos.
- Event sourcing.

## Qué no agregar ahora

- Otro framework frontend solo para engordar el CV.
- Cinco proveedores cloud.
- Tres bases NoSQL sin proyectos reales.
- Kubernetes antes de dominar containers, despliegue y observabilidad.
- Patrones arquitectónicos sin problema que resolver.
- Certificaciones como sustituto de una aplicación funcionando.

## Orden recomendado

```text
Angular + .NET + SQL
        ↓
Testing + seguridad
        ↓
Docker + CI/CD
        ↓
Azure + observabilidad
        ↓
Redis + mensajería + PostgreSQL
        ↓
AKS / MongoDB / Kafka según vacantes
```

El orden responde a dependencia y capacidad de demostración. Una API probada, containerizada, desplegada y observable vale más que una lista de servicios sin integración.

## Señal de avance

Pasar al siguiente nivel cuando exista evidencia en [[11 - Proyecto de portafolio demostrable]], no cuando se haya terminado de leer el libro correspondiente.

