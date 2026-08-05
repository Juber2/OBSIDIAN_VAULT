---
tags:
  - dotnet
  - Basico
  - csharp
Creado: 2026-05-29
Nivel: Basico
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - CSharp y Runtime dotNET
  - NET
---

# LINQ colecciones y consultas

## LINQ mental model

LINQ permite describir transformaciones sobre secuencias. En memoria usa IEnumerable; en EF Core puede traducirse a SQL con IQueryable. Esa diferencia es crucial.

## Operadores clave

`Where`, `Select`, `Any`, `All`, `FirstOrDefault`, `GroupBy`, `OrderBy`, `ToList`.

## Ejemplo en memoria


```csharp
var activeAdmins = users
    .Where(u => u.IsActive && u.Role == Role.Admin)
    .OrderBy(u => u.LastName)
    .Select(u => new UserListItem(u.Id, u.FullName))
    .ToList();
```


## IQueryable

No llames `ToList()` antes de terminar filtros y paginacion cuando consultas base de datos. Si materializas temprano, mueves trabajo innecesario a memoria.

## Errores comunes

- Usar LINQ ilegible de 20 lineas
- Materializar antes de filtrar
- No pensar en SQL generado

## Practica

- [ ] Reescribir una consulta imperativa con LINQ

## Relacionado

- [[CSharp y Runtime dotNET]]
- [[NET - Roadmap]]
