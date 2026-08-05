---
tags:
  - CSharp
  - Reflection
Creado: 2026-05-21
Relacionado:
  - CSharp Avanzado
---

# Reflection y Attributes

Reflection permite inspeccionar tipos, propiedades, métodos y atributos en runtime.

```csharp
var type = typeof(User);
var properties = type.GetProperties();
```

## Attributes

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class AuditableAttribute : Attribute {}
```

## Usos reales

- Serializadores.
- ORMs.
- Validadores.
- Frameworks de DI.
- Scanning de assemblies.

## Cuidado

Reflection puede ser costoso y romper encapsulación. En hot paths conviene cachear metadatos o usar source generators.

## Checklist

- [ ] Sé inspeccionar tipos y atributos.
- [ ] Cacheo reflection si se usa repetidamente.
- [ ] No dependo de strings mágicos sin necesidad.
- [ ] Considero source generators si performance importa.
