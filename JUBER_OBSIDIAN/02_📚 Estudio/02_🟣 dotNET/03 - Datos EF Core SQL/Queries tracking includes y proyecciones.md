---
tags:
  - dotnet
  - Intermedio
  - efcore
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Datos EF Core SQL
  - NET
---

# Queries tracking includes y proyecciones

## Tracking

EF trackea entidades para detectar cambios. Para lecturas puras usa `AsNoTracking`.

## Proyecciones

Proyecta a DTO desde query para traer solo columnas necesarias.

## Ejemplo


```csharp
var users = await db.Users
    .AsNoTracking()
    .Where(u => u.IsActive)
    .Select(u => new UserListItem(u.Id, u.Name, u.Email))
    .ToListAsync(ct);
```


## Include

Úsalo cuando necesitas grafo. Para listados, muchas veces una proyección es mejor.

## Errores comunes

- N+1 por lazy loading
- Include gigante para pantallas simples
- ToList antes de paginar

## Practica

- [ ] Optimizar listado con proyección y AsNoTracking

## Relacionado

- [[Datos EF Core SQL]]
- [[NET - Roadmap]]
