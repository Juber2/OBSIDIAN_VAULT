---
tags:
  - NET
  - Identity
Creado: 2026-05-21
Relacionado:
  - Seguridad en .NET
---

# ASP.NET Core Identity

Identity provee usuarios, passwords, roles, tokens y persistencia para autenticación.

## Incluye

- Hash de passwords.
- Lockout.
- Confirmación de email.
- Roles.
- Tokens de recuperación.
- Integración con EF Core.

## Cuidado

Identity es potente, pero puede ser pesado si solo necesitas validar tokens emitidos por otro proveedor.

## Checklist

- [ ] Entiendo cuándo usar Identity.
- [ ] Configuro password policy.
- [ ] No guardo passwords en texto plano.
- [ ] Separó autenticación local de proveedores externos.
