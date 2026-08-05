---
tags:
  - NET
  - Testing
Creado: 2026-05-21
Relacionado:
  - Testing en .NET
---

# FluentAssertions

FluentAssertions hace asserts más expresivos.

```csharp
result.Should().NotBeNull();
result.Name.Should().Be("Product A");
result.Items.Should().HaveCount(3);
```

## Excepciones

```csharp
Action act = () => service.Create(invalidRequest);
act.Should().Throw<ValidationException>();
```

## Checklist

- [ ] Los asserts se leen como intención.
- [ ] Verifico propiedades importantes.
- [ ] No sobrevalido detalles accidentales.
- [ ] Mantengo mensajes claros.
