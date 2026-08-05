---
tags:
  - NET
  - Arquitectura
Creado: 2026-05-21
Relacionado:
  - Arquitectura .NET
---

# Capas y Separación de Responsabilidades

Una API .NET mantenible separa HTTP, aplicación, dominio e infraestructura.

| Capa | Responsabilidad |
|---|---|
| API | endpoints, auth, status codes |
| Application | casos de uso |
| Domain | reglas de negocio |
| Infrastructure | EF Core, archivos, servicios externos |

```csharp
public sealed class CreateOrderHandler
{
    public async Task<OrderId> Handle(CreateOrderCommand command, CancellationToken ct)
    {
        var order = Order.Create(command.UserId, command.Items);
        await repository.SaveAsync(order, ct);
        return order.Id;
    }
}
```

## Checklist

- [ ] API no persiste directamente.
- [ ] Application orquesta.
- [ ] Domain protege reglas.
- [ ] Infrastructure implementa detalles.
