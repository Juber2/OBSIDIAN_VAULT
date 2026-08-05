---
tags:
  - CSharp
  - Delegates
Creado: 2026-05-21
Relacionado:
  - CSharp Intermedio
---

# Delegates, Events, Func y Action

Un delegate representa una referencia a un método.

```csharp
Func<int, int, int> sum = (a, b) => a + b;
Action<string> log = message => Console.WriteLine(message);
```

## Events

```csharp
public event EventHandler<OrderCreatedEventArgs>? OrderCreated;
```

Los eventos sirven para notificar algo ocurrido sin acoplar directamente al consumidor.

## Uso real

- Callbacks.
- Eventos de dominio.
- Pipelines.
- Programación funcional ligera.

## Checklist

- [ ] Entiendo diferencia entre `Func` y `Action`.
- [ ] Uso eventos para notificaciones.
- [ ] Evito eventos cuando un servicio explícito es más claro.
- [ ] No abuso de delegates si reduce legibilidad.
