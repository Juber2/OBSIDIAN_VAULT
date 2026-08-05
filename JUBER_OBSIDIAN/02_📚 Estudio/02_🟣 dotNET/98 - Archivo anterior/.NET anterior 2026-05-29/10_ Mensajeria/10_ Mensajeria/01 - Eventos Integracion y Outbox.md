---
tags:
  - NET
  - Eventos
Creado: 2026-05-21
Relacionado:
  - Mensajeria .NET
---

# Eventos, Integración y Outbox

Un evento comunica algo que ya ocurrió.

```csharp
public sealed record OrderPaid(Guid OrderId, DateTime OccurredAt);
```

## Outbox

Outbox guarda eventos en la misma transacción que el cambio de base de datos. Luego un worker publica esos eventos.

## Checklist

- [ ] Los eventos están en pasado.
- [ ] Los consumidores son idempotentes.
- [ ] Uso outbox para evitar perder eventos.
- [ ] Manejo dead-letter/retries.
