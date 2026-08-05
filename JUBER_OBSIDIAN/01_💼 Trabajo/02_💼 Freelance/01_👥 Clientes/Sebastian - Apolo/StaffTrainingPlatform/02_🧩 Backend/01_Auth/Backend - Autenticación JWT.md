---
tags:
  - backend
  - jwt
  - autenticacion
  - aspnet-core
Actualizado: 2026-05-22
---

# Backend - Autenticación JWT

## Objetivo

Autenticar al usuario mediante `POST /api/v1/auth/login` y retornar tokens que permitan al backend resolver identidad, rol, contexto multi-tenant y renovación de sesión.

## Endpoint público

```http
POST /api/v1/auth/login
```

Este debe ser el único endpoint público de autenticación inicial.

## Estrategia de tokens

| Token | Vida | Uso |
|---|---|---|
| Access Token | 15 minutos | Viaja en cada request protegido. |
| Refresh Token | 7 días | Solo se usa para renovar el access token. |

## Endpoint de refresh

```http
POST /api/v1/auth/refresh-token
```

La API no refresca tokens por iniciativa propia. Solo responde cuando el cliente llama explícitamente al endpoint de refresh.

## Validaciones de login

- Credenciales correctas.
- Usuario activo.
- Cuenta no bloqueada o inhabilitada.
- Tenant activo si el usuario pertenece a un tenant.
- Rate limiting por IP o por usuario.

## Claims mínimos

| Claim | Uso |
|---|---|
| `sub` | Identificador del usuario. |
| `email` | Email autenticado. |
| `userType` | Define si el usuario es `platform` o `tenant`. |
| `tenantId` | Tenant operativo. Solo aplica para usuarios tenant. |
| `role` | Rol usado por policies de autorización. |
| `iat` | Fecha de emisión. |
| `exp` | Fecha de expiración. |

## Reglas importantes

- No confiar en `tenantId` enviado desde el body.
- No usar flags como `typeContext` para cambiar comportamiento.
- No usar IDs mágicos como `tenantId = 0`.
- La autorización debe evaluarse en backend.
- Los controllers deben asumir el contexto desde claims.
- El refresh token debe persistirse en base de datos para permitir revocación.
- El `401` debe devolver códigos diferenciados para que el cliente sepa cómo reaccionar.

## Códigos de error de autenticación

| Código | Significado | Acción esperada del cliente |
|---|---|---|
| `TOKEN_EXPIRED` | Access token vencido. | Intentar refresh. |
| `TOKEN_INVALID` | Token manipulado o inválido. | Ir al login. |
| `SESSION_EXPIRED` | Refresh token vencido. | Ir al login. |
| `SESSION_REVOKED` | Sesión revocada. | Ir al login. |

Ejemplo:

```json
{
  "success": false,
  "errors": ["TOKEN_EXPIRED"]
}
```

## Revocación del refresh token

El servidor invalida el refresh token cuando:

- El usuario hace logout explícito.
- El usuario cambia su contraseña.
- Un admin bloquea la cuenta.
- Se detecta actividad sospechosa.

## Refresh token rotation

Cada refresh exitoso debe:

1. Validar que el refresh token esté activo.
2. Generar un nuevo access token.
3. Generar un nuevo refresh token.
4. Marcar el refresh token anterior como reemplazado.
5. Guardar en `ReplacedByToken` el nuevo token.

Si se intenta usar un refresh token ya reemplazado, se considera reuso sospechoso.

Acción:

- Revocar todos los tokens activos del usuario.
- Forzar re-login.
- Registrar evento de seguridad.

## Contexto de seguridad

Al crear un refresh token se debe guardar:

- IP.
- País derivado de la IP.
- User-Agent.
- Fecha del login original.

Cuando se use el refresh token, el backend debe comparar el contexto actual contra el contexto original.

Señales sospechosas:

- País diferente.
- Dispositivo radicalmente distinto.
- Geolocalización imposible.
- Sesiones concurrentes desde ubicaciones lejanas.
- Rate limiting excedido.

## Notificación por email

Ante uso sospechoso no crítico, el refresh token puede pasar a `pending_approval`.

El usuario recibe un email con opciones para:

- Aprobar el uso.
- Revocar la sesión.

Si no responde dentro del tiempo definido, se revoca por precaución.

El email debe obtenerse desde `User`, no desde `Person`.

## Ejemplo conceptual de extracción de contexto

```csharp
public sealed class CurrentUserContext
{
    public Guid UserId { get; init; }
    public string Email { get; init; } = string.Empty;
    public string UserType { get; init; } = string.Empty;
    public Guid? TenantId { get; init; }
    public string Role { get; init; } = string.Empty;
}
```

## Pendientes técnicos

- Definir claims exactos como constantes.
- Definir policies: `RequirePlatformUser`, `RequireTenantUser`, `RequireRole`.
- Definir manejo de errores: `401 Unauthorized` vs `403 Forbidden`.

## Notas relacionadas

- [[ADR-002 - Manejo de Access Token y Refresh Token]]
- [[ADR-003 - Seguridad Avanzada de JWT y Refresh Tokens]]
