---
tags:
  - EFCore
  - Relaciones
Creado: 2026-05-21
Relacionado:
  - Entity Framework Core
---

# Relaciones, Tracking e Includes

EF Core puede cargar relaciones con `Include`, lazy loading o proyecciones.

```csharp
var order = await db.Orders
    .Include(o => o.Lines)
    .FirstOrDefaultAsync(o => o.Id == id, ct);
```

## Tracking

Tracking permite detectar cambios para guardar después. Para lecturas, usa `AsNoTracking`.

```csharp
var users = await db.Users
    .AsNoTracking()
    .ToListAsync(ct);
```

## Checklist

- [ ] Uso `AsNoTracking` en lecturas.
- [ ] Evito includes innecesarios.
- [ ] Entiendo lazy loading y sus riesgos.
- [ ] Reviso N+1.
