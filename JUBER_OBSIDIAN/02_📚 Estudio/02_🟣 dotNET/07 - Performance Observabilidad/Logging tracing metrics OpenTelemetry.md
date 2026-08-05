---
tags:
  - dotnet
  - Senior
  - performance
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Performance Observabilidad dotNET
  - NET
---

# Logging tracing metrics OpenTelemetry

## Logging estructurado

Loguea propiedades, no strings concatenados. Permite filtrar por UserId, TenantId, CorrelationId y Operation.

## Tracing

Muestra camino de una request entre API, DB, HTTP externo y colas.

## Metrics

Mide latencia, throughput, errores, colas y recursos.

## Ejemplo log


```csharp
logger.LogInformation("User {UserId} created in tenant {TenantId}", userId, tenantId);
```


## Errores comunes

- Loguear tokens o passwords
- Logs sin contexto
- No distinguir info/warn/error

## Practica

- [ ] Agregar CorrelationId a una request

## Relacionado

- [[Performance Observabilidad dotNET]]
- [[NET - Roadmap]]
