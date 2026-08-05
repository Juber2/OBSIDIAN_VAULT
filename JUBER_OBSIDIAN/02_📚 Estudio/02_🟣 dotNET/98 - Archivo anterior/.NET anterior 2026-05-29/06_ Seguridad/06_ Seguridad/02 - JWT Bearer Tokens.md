---
tags:
  - NET
  - JWT
Creado: 2026-05-21
Relacionado:
  - Seguridad en .NET
---

# JWT Bearer Tokens

JWT permite transportar claims firmados. No es una sesión; es un token que debe validarse.

```csharp
builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true
        };
    });
```

## Reglas

- Expiración corta.
- Firma fuerte.
- Validar issuer y audience.
- No guardar secretos débiles.
- No meter datos sensibles en payload.

## Checklist

- [ ] Valido firma.
- [ ] Valido expiración.
- [ ] Uso HTTPS.
- [ ] Tengo estrategia de refresh/revocación si aplica.
