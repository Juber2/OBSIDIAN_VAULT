---
tags:
  - dotnet
  - laboratorio
  - aspnetcore
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos dotNET
  - NET
---

# Laboratorio 02 - API CRUD ASP.NET Core

## Objetivo

Construir una API CRUD con controllers, rutas versionadas, DTOs, validation, ProblemDetails y OpenAPI.

## Endpoints

- `GET /api/v1/users` con paginacion.
- `GET /api/v1/users/{id}`.
- `POST /api/v1/users`.
- `PATCH /api/v1/users/{id}`.
- `DELETE /api/v1/users/{id}`.

## Reglas

No retornar entidades internas. No usar flags magicos en body. Usar status codes correctos: 200, 201, 204, 400, 404, 409.

## Ejemplo DTO


```csharp
public sealed record CreateUserRequest(string Email, string Name);
public sealed record UserResponse(Guid Id, string Email, string Name);
```


## Errores comunes

- Controller con logica de negocio
- Devolver 200 para errores
- No validar email duplicado

## Practica

- [ ] Implementar endpoints
- [ ] Probar con archivo .http
- [ ] Documentar contrato

## Relacionado

- [[Laboratorios Practicos dotNET]]
- [[NET]]
