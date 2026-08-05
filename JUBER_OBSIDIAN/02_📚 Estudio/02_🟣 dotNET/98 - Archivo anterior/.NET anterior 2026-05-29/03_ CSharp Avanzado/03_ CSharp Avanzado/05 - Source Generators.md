---
tags:
  - CSharp
  - SourceGenerators
Creado: 2026-05-21
Relacionado:
  - CSharp Avanzado
---

# Source Generators

Source generators generan código en compilación. Ayudan a evitar reflection en runtime y reducir boilerplate.

## Usos

- Serialización.
- Mappers.
- Validadores.
- Código repetitivo basado en atributos.

## Ejemplo conceptual

```csharp
[GenerateMapper]
public partial class UserMapper {}
```

El generator puede crear parte de la clase antes de compilar.

## Checklist

- [ ] Entiendo que corren en compilación.
- [ ] Los uso para boilerplate real.
- [ ] No complico el build sin beneficio claro.
- [ ] Reviso código generado cuando falla.
