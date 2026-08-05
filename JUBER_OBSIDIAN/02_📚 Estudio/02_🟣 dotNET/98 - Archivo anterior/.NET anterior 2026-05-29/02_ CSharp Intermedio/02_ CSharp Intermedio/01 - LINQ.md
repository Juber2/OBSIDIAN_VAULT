---
tags:
  - CSharp
  - LINQ
Creado: 2026-05-21
Relacionado:
  - CSharp Intermedio
---

# LINQ

LINQ permite consultar colecciones con operadores expresivos.

```csharp
var activeUsers = users
    .Where(u => u.IsActive)
    .OrderBy(u => u.Name)
    .Select(u => new UserDto(u.Id, u.Name))
    .ToList();
```

## Operadores clave

| Operador | Uso |
|---|---|
| `Where` | filtrar |
| `Select` | proyectar |
| `Any` | existe al menos uno |
| `All` | todos cumplen |
| `GroupBy` | agrupar |
| `ToList` | materializar |

## Ojo con EF Core

En `IQueryable`, LINQ se traduce a SQL. En `IEnumerable`, se ejecuta en memoria.

## Checklist

- [ ] Diferencio `IEnumerable` e `IQueryable`.
- [ ] No llamo `ToList` demasiado temprano.
- [ ] Proyecto solo campos necesarios.
- [ ] Reviso SQL generado cuando importa.
