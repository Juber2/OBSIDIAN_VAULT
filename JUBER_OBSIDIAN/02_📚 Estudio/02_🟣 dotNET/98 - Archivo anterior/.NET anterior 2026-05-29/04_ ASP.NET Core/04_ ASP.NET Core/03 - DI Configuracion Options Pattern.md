---
tags:
  - ASPNETCore
  - DI
Creado: 2026-05-21
Relacionado:
  - ASP.NET Core
---

# DI, Configuración y Options Pattern

ASP.NET Core trae inyección de dependencias integrada.

```csharp
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddSingleton<IClock, SystemClock>();
```

## Lifetimes

| Lifetime | Uso |
|---|---|
| Singleton | una instancia para toda la app |
| Scoped | una instancia por request |
| Transient | nueva instancia cada vez |

## Options Pattern

```csharp
builder.Services.Configure<JwtOptions>(
    builder.Configuration.GetSection("Jwt"));
```

## Checklist

- [ ] No inyecto scoped dentro de singleton.
- [ ] Uso interfaces cuando hay frontera real.
- [ ] Configuro options tipadas.
- [ ] Valido configuración crítica al iniciar.
