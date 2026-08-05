---
tags:
  - NET
  - Hexagonal
Creado: 2026-05-21
Relacionado:
  - Arquitectura .NET
---

# Arquitectura Hexagonal en .NET

Hexagonal organiza el sistema por puertos y adaptadores.

## Puertos

```csharp
public interface IPaymentGateway
{
    Task<PaymentResult> ChargeAsync(Money amount, CancellationToken ct);
}
```

## Adaptador

```csharp
public sealed class StripePaymentGateway : IPaymentGateway
{
    public Task<PaymentResult> ChargeAsync(Money amount, CancellationToken ct)
    {
        // llamada externa
    }
}
```

## Checklist

- [ ] El core no conoce proveedores externos.
- [ ] Los adapters traducen modelos.
- [ ] Puedo testear con fakes.
- [ ] Los puertos expresan negocio.
