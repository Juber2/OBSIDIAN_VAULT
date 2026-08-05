---
tags:
  - NET
  - CQRS
  - MediatR
Creado: 2026-05-21
Relacionado:
  - Arquitectura .NET
---

# CQRS y MediatR

CQRS separa comandos de consultas. MediatR ayuda a enviar requests a handlers.

```csharp
public sealed record CreateProductCommand(string Name, decimal Price)
    : IRequest<Guid>;

public sealed class CreateProductHandler
    : IRequestHandler<CreateProductCommand, Guid>
{
    public async Task<Guid> Handle(CreateProductCommand request, CancellationToken ct)
    {
        // crear producto
    }
}
```

## Pipeline behaviors

MediatR permite behaviors para validación, logging, transacciones o métricas.

## Checklist

- [ ] Commands cambian estado.
- [ ] Queries retornan DTOs.
- [ ] No uso MediatR para esconder mal diseño.
- [ ] Los handlers son pequeños y testeables.
