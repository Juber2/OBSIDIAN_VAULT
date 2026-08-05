---
tags:
  - CSharp
  - PatternMatching
Creado: 2026-05-21
Relacionado:
  - CSharp Avanzado
---

# Pattern Matching Avanzado

Pattern matching permite expresar decisiones por forma, tipo y valores.

```csharp
string Describe(object value) => value switch
{
    null => "null",
    int n when n > 0 => "positive int",
    string { Length: > 10 } => "long string",
    User { IsActive: true } user => $"active {user.Name}",
    _ => "unknown"
};
```

## Beneficio

Reduce condicionales verbosos y ayuda a modelar estados.

## Checklist

- [ ] Uso patterns para mejorar claridad.
- [ ] Evito switches gigantes con lógica compleja.
- [ ] Combino con records cuando modela estados.
- [ ] Mantengo casos default seguros.
