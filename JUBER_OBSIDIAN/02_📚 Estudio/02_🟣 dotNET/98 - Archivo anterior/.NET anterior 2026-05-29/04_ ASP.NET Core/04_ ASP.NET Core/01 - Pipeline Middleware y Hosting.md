---
tags:
  - ASPNETCore
  - Middleware
Creado: 2026-05-21
Relacionado:
  - ASP.NET Core
---

# Pipeline, Middleware y Hosting

ASP.NET Core procesa requests mediante un pipeline de middleware.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

## Middleware

Cada middleware puede ejecutar lógica antes y después del siguiente.

## Orden importa

Autenticación debe ir antes de autorización. Manejo de errores debe estar temprano en el pipeline.

## Checklist

- [ ] Entiendo el orden del pipeline.
- [ ] Sé crear middleware propio.
- [ ] Uso environment para configurar comportamiento.
- [ ] No meto lógica de negocio en middleware.
