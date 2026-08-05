---
tags:
  - CSharp
  - Nullability
Creado: 2026-05-21
Relacionado:
  - .NET Fundamentos
---

# Nullable Reference Types

Nullable Reference Types ayudan a detectar posibles `NullReferenceException` desde compilación.

```csharp
string name = "Juber";
string? nickname = null;
```

## Reglas

- `string` significa que no debería ser null.
- `string?` significa que puede ser null.
- El compilador advierte cuando usas algo posiblemente null.

```csharp
if (nickname is not null)
{
    Console.WriteLine(nickname.Length);
}
```

## Checklist

- [ ] Activo nullable en proyectos nuevos.
- [ ] Uso `?` solo cuando realmente puede faltar.
- [ ] Valido entrada externa.
- [ ] Evito `!` salvo casos justificados.
