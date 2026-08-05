---
tags:
  - NET
  - CleanArchitecture
Creado: 2026-05-21
Relacionado:
  - Arquitectura .NET
---

# Clean Architecture en .NET

Clean Architecture mantiene el negocio independiente de frameworks.

## Estructura

```text
src/
  MyApp.Domain/
  MyApp.Application/
  MyApp.Infrastructure/
  MyApp.Api/
tests/
```

## Dependencias

API depende de Application. Infrastructure implementa interfaces de Application. Domain no depende de nadie.

```csharp
public interface IUserRepository
{
    Task<User?> FindByEmailAsync(Email email, CancellationToken ct);
    Task SaveAsync(User user, CancellationToken ct);
}
```

## Checklist

- [ ] Domain no referencia EF Core.
- [ ] Application no referencia ASP.NET Core.
- [ ] Infrastructure contiene adapters.
- [ ] Tests de dominio no levantan web host.
