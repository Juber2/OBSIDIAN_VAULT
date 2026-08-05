---
tags:
  - NET
  - Authorization
Creado: 2026-05-21
Relacionado:
  - Seguridad en .NET
---

# Roles, Policies y Resource-Based Authorization

Los roles son simples, pero las policies expresan mejor permisos complejos.

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("CanManageProducts", policy =>
        policy.RequireClaim("permission", "products.manage"));
});
```

## Resource-based authorization

Valida si el usuario puede acceder a un recurso específico, no solo si tiene rol.

```csharp
if (order.UserId != currentUser.Id)
{
    return Forbid();
}
```

## Checklist

- [ ] No uso solo roles para todo.
- [ ] Valido ownership del recurso.
- [ ] Uso policies para permisos.
- [ ] Mantengo permisos auditables.
