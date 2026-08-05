---
tags:
  - ASPNETCore
  - Validacion
Creado: 2026-05-21
Relacionado:
  - ASP.NET Core
---

# Validaciones y FluentValidation

Validar entrada evita que datos inválidos lleguen al dominio.

```csharp
public record CreateProductRequest(string Name, decimal Price);

public class CreateProductValidator : AbstractValidator<CreateProductRequest>
{
    public CreateProductValidator()
    {
        RuleFor(x => x.Name).NotEmpty().MaximumLength(120);
        RuleFor(x => x.Price).GreaterThan(0);
    }
}
```

## Capas

- DTO: formato y campos requeridos.
- Caso de uso: reglas de operación.
- Dominio: invariantes que nunca deben romperse.

## Checklist

- [ ] Valido entrada externa.
- [ ] No duplico reglas sin intención.
- [ ] El dominio sigue protegiéndose.
- [ ] Los errores son claros para el consumidor.
