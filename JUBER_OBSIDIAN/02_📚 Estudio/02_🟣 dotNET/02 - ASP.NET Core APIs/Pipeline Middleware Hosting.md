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

# Pipeline Middleware Hosting

## Pipeline

Cada request pasa por middleware en orden. El orden importa: routing, auth, authorization, endpoints, exception handling y static files pueden cambiar comportamiento.

## Program.cs moderno


```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
builder.Services.AddProblemDetails();

var app = builder.Build();
app.UseExceptionHandler();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```


## Hosting

WebApplication combina host, DI, config y pipeline. Entenderlo evita Program.cs gigante e inicializacion accidental.

## Errores comunes

- Registrar middleware en orden incorrecto
- Meter llamadas externas en Program.cs
- No diferenciar service registration y request pipeline

## Practica

- [ ] Dibujar pipeline de una API segura

## Relacionado

- [[ASP.NET Core APIs]]
- [[NET - Roadmap]]
