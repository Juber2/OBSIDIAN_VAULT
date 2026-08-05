---
tags:
  - dotnet
  - Basico
  - csharp
Creado: 2026-05-29
Nivel: Basico
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - CSharp y Runtime dotNET
  - NET
---

# CSharp moderno tipos records pattern matching

## Tipos que importan

C# moderno favorece tipos expresivos: records para datos inmutables, enums cuando hay conjunto cerrado, value objects cuando una regla de negocio merece tipo propio.

## Records

Un record es ideal para DTOs internos, mensajes y valores inmutables. No lo uses como entidad EF por defecto sin entender tracking e identidad.

## Pattern matching

Permite expresar decisiones por forma, tipo o valor sin ifs largos.

## Ejemplo


```csharp
public sealed record Money(decimal Amount, string Currency);

static string Describe(OrderStatus status) => status switch
{
    OrderStatus.Pending => "Pendiente",
    OrderStatus.Paid => "Pagada",
    OrderStatus.Cancelled => "Cancelada",
    _ => "Desconocida"
};
```


## Errores comunes

- Usar record para todo
- Enums sin caso Unknown
- Switchs sin default razonable

## Practica

- [ ] Modelar UserRole, Email y Money

## Relacionado

- [[CSharp y Runtime dotNET]]
- [[NET - Roadmap]]
