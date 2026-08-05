---
tags:
  - dotnet
  - Avanzado
  - arquitectura
Creado: 2026-05-29
Nivel: Avanzado
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Arquitectura dotNET
  - NET
---

# DDD tactico entidades value objects agregados

## Entidad

Tiene identidad y ciclo de vida. Ejemplo: User, Course, Tenant.

## Value Object

No tiene identidad, se compara por valor y protege invariantes. Ejemplo: Email, Money, DateRange.

## Agregado

Conjunto de objetos que mantiene reglas consistentes bajo una raiz. No todo modelo necesita agregado complejo.

## Ejemplo value object


```csharp
public sealed record Email
{
    public string Value { get; }
    public Email(string value)
    {
        if (!value.Contains("@")) throw new DomainException("Invalid email");
        Value = value;
    }
}
```


## Errores comunes

- Anemizar dominio por completo
- Crear agregados enormes
- Value objects sin reglas

## Practica

- [ ] Modelar Tenant y User como agregados simples

## Relacionado

- [[Arquitectura dotNET]]
- [[NET - Roadmap]]
