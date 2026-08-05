---
tags:
  - EFCore
  - LINQ
Creado: 2026-05-21
Relacionado:
  - Entity Framework Core
---

# LINQ to Entities y Queries

EF Core traduce LINQ a SQL cuando trabajas con `IQueryable`.

```csharp
var products = await db.Products
    .Where(p => p.IsActive)
    .OrderBy(p => p.Name)
    .Select(p => new ProductDto(p.Id, p.Name, p.Price))
    .ToListAsync(ct);
```

## Buenas prácticas

- Proyecta a DTO.
- Usa paginación.
- Revisa SQL generado.
- Evita lógica C# que no se puede traducir.

## Checklist

- [ ] Uso `ToListAsync`.
- [ ] Proyecto solo columnas necesarias.
- [ ] No materializo antes de filtrar.
- [ ] Leo logs de queries cuando hay lentitud.
