---
tags:
  - CSharp
  - Records
Creado: 2026-05-21
Relacionado:
  - CSharp Intermedio
---

# Records, Tuples y Value Objects

Records son útiles para datos inmutables y comparación por valor.

```csharp
public record Email(string Value)
{
    public Email : this(Value.Trim().ToLowerInvariant())
    {
        if (!Value.Contains("@")) throw new ArgumentException("Invalid email");
    }
}
```

## Tuples

```csharp
(bool IsValid, string? Error) Validate(User user)
{
    return user.Email.Contains("@")
        ? (true, null)
        : (false, "Invalid email");
}
```

## Recomendación

Usa tuples para resultados locales simples. Usa records cuando el concepto tiene nombre y vive más tiempo.

## Checklist

- [ ] Uso records para DTOs o value objects.
- [ ] No abuso de tuples en APIs públicas.
- [ ] Mantengo invariantes dentro del value object.
- [ ] Entiendo igualdad por valor.
