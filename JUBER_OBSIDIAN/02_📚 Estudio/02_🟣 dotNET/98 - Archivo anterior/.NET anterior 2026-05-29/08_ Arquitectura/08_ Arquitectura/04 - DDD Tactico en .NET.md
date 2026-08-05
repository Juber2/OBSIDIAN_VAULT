---
tags:
  - NET
  - DDD
Creado: 2026-05-21
Relacionado:
  - Arquitectura .NET
---

# DDD Táctico en .NET

DDD ayuda a modelar reglas del negocio con lenguaje claro.

## Building blocks

| Concepto | Ejemplo |
|---|---|
| Entity | `Order` |
| Value Object | `Email`, `Money` |
| Aggregate | `Order` con líneas |
| Repository | `IOrderRepository` |
| Domain Event | `OrderPaid` |

```csharp
public sealed record Email
{
    public string Value { get; }

    public Email(string value)
    {
        if (!value.Contains("@")) throw new ArgumentException("Invalid email");
        Value = value;
    }
}
```

## Checklist

- [ ] Uso value objects para valores con reglas.
- [ ] Protejo invariantes en agregados.
- [ ] Evito anémicos si hay lógica real.
- [ ] El código usa lenguaje del negocio.
