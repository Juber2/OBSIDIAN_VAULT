---
tags:
  - adr
  - jwt
  - api
  - multi-tenant
  - autenticacion
Estado: Aceptada
Fecha: 2026-05-22
---

# ADR-001 - Autenticación JWT y API Multi-Tenant

## Estado

Aceptada.

## Contexto

StaffTrainingPlatform necesita autenticar usuarios desde el frontend contra un backend ASP.NET Core y operar bajo un modelo multi-tenant.

El sistema tiene dos contextos principales:

- **platform**: administradores globales de la plataforma, no pertenecen a ningún tenant.
- **tenant**: usuarios que operan dentro de un tenant específico.

## Decisión

Se usará **JWT** como mecanismo de autenticación entre frontend y backend.

La API separará explícitamente los endpoints por contexto:

- `/api/v1/platform/...`
- `/api/v1/tenant/...`

El tenant operativo se obtendrá desde el JWT, nunca desde el body del request.

## Endpoints

### Auth público

```http
POST /api/v1/auth/login
```

### Tenant

```http
GET    /api/v1/tenant/users
POST   /api/v1/tenant/users
GET    /api/v1/tenant/users/{userId}
PATCH  /api/v1/tenant/users/{userId}
DELETE /api/v1/tenant/users/{userId}
```

### Platform

```http
GET    /api/v1/platform/users
POST   /api/v1/platform/users
GET    /api/v1/platform/users/{userId}
PATCH  /api/v1/platform/users/{userId}
DELETE /api/v1/platform/users/{userId}
GET    /api/v1/platform/tenants/{tenantId}/users
```

## Reglas de diseño

- La ruta identifica el recurso.
- El JWT valida el acceso, nunca la ruta por sí sola.
- El tenant siempre se obtiene desde el JWT, nunca desde el body.
- Query params solo para filtros, paginación y ordenamiento.
- Login es el único endpoint público.
- No usar flags ni IDs mágicos en el body para cambiar comportamiento.

## Naming

### Controllers

- `TenantUsersController`
- `PlatformUsersController`
- `TenantEmployeesController`

Regla:

```text
{contexto}{recurso}Controller
```

### DTOs / Requests

- `CreateTenantUserRequest`
- `UpdateTenantUserRequest`
- `CreatePlatformUserRequest`
- `UpdatePlatformUserRequest`

Regla:

```text
{operación}{contexto}{recurso}Request
```

## Organización de carpetas sugerida

```text
src/
  modules/
    tenant/
      users/
        requests/
          CreateTenantUserRequest.cs
          UpdateTenantUserRequest.cs
        responses/
          TenantUserResponse.cs
        TenantUsersController.cs
        TenantUsersService.cs
    platform/
      users/
        requests/
          CreatePlatformUserRequest.cs
          UpdatePlatformUserRequest.cs
        responses/
          PlatformUserResponse.cs
        PlatformUsersController.cs
        PlatformUsersService.cs
```

## Serialización JSON

```csharp
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    });
```

Reglas:

- C# usa PascalCase: `UserName`.
- JSON usa camelCase: `userName`.
- No es necesario `[JsonPropertyName]` salvo casos especiales como snake_case, legacy o APIs externas.

## JWT Claims

### Identidad

| Claim | Valor |
|---|---|
| `sub` | userId |
| `email` | email del usuario |
| `userType` | `platform` o `tenant` |
| `tenantId` | solo si `userType` es `tenant` |

### Autorización

| Claim | Valor |
|---|---|
| `role` | `SuperAdmin`, `CompanyAdmin` o `Employee` |

### Estándar

| Claim | Valor |
|---|---|
| `iat` | fecha de emisión |
| `exp` | fecha de expiración |

## Flujo de autenticación

1. Frontend envía credenciales al `POST /api/v1/auth/login`.
2. Backend valida credenciales, cuenta activa y estado.
3. Backend genera y retorna el JWT.
4. Frontend almacena el token y lo envía en cada request vía `Authorization: Bearer {token}`.
5. Backend valida el token y extrae el contexto en cada request.

## Protecciones en login

- Rate limiting por IP o por usuario.
- Validación de credenciales.
- Validación de cuenta activa.

## Decisiones descartadas

| Opción | Razón |
|---|---|
| `tenantId` en body | Viola separación de contexto. |
| `tenantId = 0` como flag de admin | Comportamiento condicional frágil. |
| `typeContext` en body | Flag disfrazado. |
| `UsersTenantController` | Naming invertido. |
| `[JsonPropertyName]` por defecto | Innecesario con política global. |
| `clientId` en login | Over-engineering sin caso de uso actual. |

## Consecuencias

- Los endpoints quedan más predecibles.
- La autorización depende de claims y policies, no de datos enviados por el cliente.
- El backend puede proteger mejor los recursos tenant.
- El frontend no necesita conocer ni enviar `tenantId` para operar dentro del tenant actual.

## Notas relacionadas

- [[Backend - Autenticación JWT]]
- [[Backend - API Multi-Tenant]]
- [[Frontend - Manejo de JWT]]
