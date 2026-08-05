---
tags:
  - dotnet
  - Avanzado
  - testing
Creado: 2026-05-29
Nivel: Avanzado
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Testing Calidad dotNET
  - NET
---

# Unit Testing xUnit FluentAssertions

## Unit test

Debe probar una regla pequeña sin infraestructura. Ideal para dominio, mappers, validators y servicios puros.

## Ejemplo


```csharp
[Fact]
public void Create_should_fail_when_email_is_invalid()
{
    var act = () => new Email("bad");
    act.Should().Throw<DomainException>();
}
```


## Naming

Nombre de test debe contar escenario y expectativa. No importa si es largo si evita ambigüedad.

## Errores comunes

- Tests sin asserts claros
- Probar getters/setters
- Depender de DateTime.Now

## Practica

- [ ] Crear tests de Email y Money

## Relacionado

- [[Testing Calidad dotNET]]
- [[NET - Roadmap]]
