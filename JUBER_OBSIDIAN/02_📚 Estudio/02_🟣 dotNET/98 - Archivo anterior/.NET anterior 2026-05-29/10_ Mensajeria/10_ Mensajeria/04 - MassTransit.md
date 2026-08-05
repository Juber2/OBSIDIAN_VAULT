---
tags:
  - NET
  - MassTransit
Creado: 2026-05-21
Relacionado:
  - Mensajeria .NET
---

# MassTransit

MassTransit abstrae mensajería sobre RabbitMQ, Azure Service Bus y otros transportes.

```csharp
public class OrderPaidConsumer : IConsumer<OrderPaid>
{
    public Task Consume(ConsumeContext<OrderPaid> context)
    {
        return Task.CompletedTask;
    }
}
```

## Beneficios

- Consumers tipados.
- Retries.
- Sagas.
- Integración con DI.

## Checklist

- [ ] Registro consumers correctamente.
- [ ] Configuro retry.
- [ ] Uso mensajes versionables.
- [ ] Evito acoplar dominio al broker.
