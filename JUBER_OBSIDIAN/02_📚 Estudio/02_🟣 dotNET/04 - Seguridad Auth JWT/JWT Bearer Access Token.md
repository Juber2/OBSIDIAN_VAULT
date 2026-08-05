---
tags:
  - dotnet
  - Intermedio
  - seguridad
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Seguridad Auth JWT dotNET
  - NET
---

# JWT Bearer Access Token

## Access token

Token corto que viaja en `Authorization: Bearer`. Debe tener vida corta porque normalmente no se consulta DB en cada request.

## Configuracion base


```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
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


## Claims recomendados

`sub` como userId, `email`, `userType`, `tenantId` si aplica, `role`, `iat`, `exp`.

## Errores 401/403

401 = no autenticado o token invalido. 403 = autenticado pero sin permisos.

## Errores comunes

- Tokens largos de 7 dias como access token
- Meter datos sensibles en JWT
- No rotar signing keys

## Practica

- [ ] Crear token de 15 minutos con claims minimos

## Relacionado

- [[Seguridad Auth JWT dotNET]]
- [[NET - Roadmap]]
