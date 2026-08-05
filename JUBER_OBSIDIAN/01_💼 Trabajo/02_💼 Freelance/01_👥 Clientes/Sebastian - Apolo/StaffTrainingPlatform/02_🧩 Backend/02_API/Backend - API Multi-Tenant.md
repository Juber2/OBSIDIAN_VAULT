---
tags:
  - backend
  - api
  - multi-tenant
  - aspnet-core
Actualizado: 2026-05-22
---

# Backend - API Multi-Tenant

## Diseño de rutas

La API separa recursos por contexto:

```text
/api/v1/tenant/...
/api/v1/platform/...
```

## Auth

```http
POST /api/v1/auth/login
```

## Tenant users

```http
GET    /api/v1/tenant/users
POST   /api/v1/tenant/users
GET    /api/v1/tenant/users/{userId}
PATCH  /api/v1/tenant/users/{userId}
DELETE /api/v1/tenant/users/{userId}
```

## Platform users

```http
GET    /api/v1/platform/users
POST   /api/v1/platform/users
GET    /api/v1/platform/users/{userId}
PATCH  /api/v1/platform/users/{userId}
DELETE /api/v1/platform/users/{userId}
GET    /api/v1/platform/tenants/{tenantId}/users
```

## Convenciones de controllers

| Recurso | Controller |
|---|---|
| Usuarios tenant | `TenantUsersController` |
| Usuarios platform | `PlatformUsersController` |
| Empleados tenant | `TenantEmployeesController` |

Regla:

```text
{contexto}{recurso}Controller
```

## Convenciones de requests

| Caso | Request |
|---|---|
| Crear usuario tenant | `CreateTenantUserRequest` |
| Actualizar usuario tenant | `UpdateTenantUserRequest` |
| Crear usuario platform | `CreatePlatformUserRequest` |
| Actualizar usuario platform | `UpdatePlatformUserRequest` |

Regla:

```text
{operación}{contexto}{recurso}Request
```

## Serialización JSON

La API debe configurar camelCase global:

```csharp
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    });
```

## Reglas de implementación

- Usar query params solo para filtros, paginación y ordenamiento.
- No enviar `tenantId` en body para operaciones tenant.
- No condicionar comportamiento usando flags de body.
- Evitar controllers genéricos cuando el contexto cambia reglas de autorización.
- Mantener requests separados entre platform y tenant aunque los campos se parezcan.

## Nota relacionada

- [[ADR-001 - Autenticación JWT y API Multi-Tenant]]
