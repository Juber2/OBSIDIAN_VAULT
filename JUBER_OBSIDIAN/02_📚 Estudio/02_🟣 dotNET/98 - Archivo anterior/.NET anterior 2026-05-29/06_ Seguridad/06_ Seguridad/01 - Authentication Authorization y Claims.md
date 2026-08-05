---
tags:
  - NET
  - Seguridad
Creado: 2026-05-21
Relacionado:
  - Seguridad en .NET
---

# Authentication, Authorization y Claims

Autenticación responde: quién eres. Autorización responde: qué puedes hacer.

## Claims

Un claim representa información del usuario.

```csharp
var userId = User.FindFirst("sub")?.Value;
var email = User.FindFirst(ClaimTypes.Email)?.Value;
```

## Pipeline

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

## Checklist

- [ ] No confundo identidad con permisos.
- [ ] Leo claims con nombres estables.
- [ ] Valido autorización por endpoint.
- [ ] No confío en datos enviados por frontend.
