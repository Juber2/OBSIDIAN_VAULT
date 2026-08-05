---
tags:
  - CSharp
  - Colecciones
Creado: 2026-05-21
Relacionado:
  - .NET Fundamentos
---

# Arrays, List, Dictionary y HashSet

Las colecciones son estructuras para trabajar con grupos de datos.

| Colección | Uso |
|---|---|
| `T[]` | tamaño fijo |
| `List<T>` | lista ordenada y dinámica |
| `Dictionary<TKey,TValue>` | búsqueda por clave |
| `HashSet<T>` | valores únicos |

```csharp
var users = new List<string> { "Ana", "Luis" };
var roles = new HashSet<string> { "Admin", "User" };
var prices = new Dictionary<string, decimal>
{
    ["P001"] = 10.5m
};
```

## Checklist

- [ ] Uso `Dictionary` para búsquedas por clave.
- [ ] Uso `HashSet` para evitar duplicados.
- [ ] Evito recorrer listas si puedo acceder por clave.
- [ ] Elijo colección según operación principal.
