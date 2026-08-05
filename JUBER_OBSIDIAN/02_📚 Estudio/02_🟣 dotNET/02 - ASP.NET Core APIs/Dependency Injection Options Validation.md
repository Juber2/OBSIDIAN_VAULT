---
tags:
  - dotnet
  - Intermedio
  - aspnetcore
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - ASP.NET Core APIs
  - NET
---

# Dependency Injection Options Validation

## DI en ASP.NET Core

El contenedor resuelve services por lifetime: Singleton, Scoped y Transient. En web, Scoped normalmente vive lo que dura el request.

## Options Pattern

Config tipada para settings. Evita leer strings sueltos en toda la app.

## Ejemplo Options


```csharp
builder.Services
    .AddOptions<JwtOptions>()
    .BindConfiguration("Jwt")
    .ValidateDataAnnotations()
    .ValidateOnStart();
```


## Validation

Valida requests antes de llegar a dominio. FluentValidation o DataAnnotations pueden servir, pero el dominio igual debe proteger invariantes críticas.

## Errores comunes

- Singleton dependiendo de Scoped
- Config hardcodeada
- Validar solo en frontend

## Practica

- [ ] Crear JwtOptions validado al arrancar

## Relacionado

- [[ASP.NET Core APIs]]
- [[NET - Roadmap]]
