---
tags:
  - ASPNETCore
  - Observabilidad
Creado: 2026-05-21
Relacionado:
  - ASP.NET Core
---

# Logging, OpenAPI y Health Checks

APIs reales deben ser consumibles y observables.

## Logging estructurado

```csharp
logger.LogInformation("Creating product {ProductId}", product.Id);
```

## OpenAPI

Swagger/OpenAPI documenta endpoints, contratos y respuestas.

```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
```

## Health checks

```csharp
builder.Services.AddHealthChecks();
app.MapHealthChecks("/health");
```

## Checklist

- [ ] Los logs tienen contexto.
- [ ] No logueo secretos.
- [ ] Swagger refleja contratos reales.
- [ ] Health check valida dependencias críticas.
