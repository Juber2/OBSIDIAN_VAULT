---
tags:
  - EFCore
  - Performance
Creado: 2026-05-21
Relacionado:
  - Entity Framework Core
---

# Performance EF Core

EF Core es productivo, pero puede generar queries caras si no se usa con cuidado.

## Riesgos

- N+1.
- `Include` excesivo.
- Tracking innecesario.
- Falta de índices.
- Paginación mal diseñada.
- Materialización temprana.

## Herramientas

```csharp
var sql = db.Products.Where(p => p.IsActive).ToQueryString();
```

## Checklist

- [ ] Uso `AsNoTracking` en lecturas.
- [ ] Proyecto DTOs.
- [ ] Reviso SQL con `ToQueryString`.
- [ ] Uso índices según consultas reales.
- [ ] Considero compiled queries en hot paths.
