---
tags:
  - frontend
  - jwt
  - auth
  - angular
Actualizado: 2026-05-22
---

# Frontend - Manejo de JWT

## Flujo

1. El frontend envía credenciales a `POST /api/v1/auth/login`.
2. Recibe un access token y un refresh token si las credenciales son válidas.
3. Guarda los tokens según la estrategia definida.
4. Envía el access token en cada request protegida:

```http
Authorization: Bearer {token}
```

5. Si el backend responde `401 TOKEN_EXPIRED`, el frontend debe intentar refresh.
6. Si el backend responde `403`, el usuario está autenticado pero no tiene permisos.

## Estrategia definida

| Token | Vida | Uso |
|---|---|---|
| Access Token | 15 minutos | Viaja en cada request. |
| Refresh Token | 7 días | Solo para renovar el access token. |

## Regla del cliente

El cliente no lee el `exp` del JWT para anticiparse.

Siempre reacciona al `401`:

1. Hace el request normal con el access token.
2. Si la API devuelve `401 TOKEN_EXPIRED`, llama a `/auth/refresh-token`.
3. Reintenta el request original con el nuevo access token.
4. Si el refresh falla, borra tokens y redirige al login.

Todo debe manejarse desde un interceptor HTTP global.

## Responsabilidades del frontend

- No enviar `tenantId` en operaciones tenant.
- No decidir permisos críticos solo en cliente.
- Ocultar opciones de UI según rol para mejorar experiencia.
- Dejar que el backend sea la autoridad final.
- Adjuntar el token con un interceptor HTTP.
- Interceptar `401`.
- Refrescar token solo cuando el backend devuelva `TOKEN_EXPIRED`.
- Borrar tokens y redirigir al login ante `TOKEN_INVALID`, `SESSION_EXPIRED` o `SESSION_REVOKED`.

## Códigos de autenticación

| Código | Acción cliente |
|---|---|
| `TOKEN_EXPIRED` | Intentar refresh. |
| `TOKEN_INVALID` | Borrar tokens e ir al login. |
| `SESSION_EXPIRED` | Borrar tokens e ir al login. |
| `SESSION_REVOKED` | Borrar tokens e ir al login. |

## Aprobación de sesión sospechosa

Si el backend detecta uso sospechoso del refresh token, puede marcar la sesión como `pending_approval` y enviar un email al usuario.

El frontend debe estar preparado para:

- Mostrar sesión expirada o pendiente si el backend no permite continuar.
- Redirigir al login cuando la sesión sea revocada.
- No intentar refrescar infinitamente cuando el backend devuelve `SESSION_REVOKED`.

## Datos derivados del token

El frontend puede usar claims para UI:

| Claim | Uso en UI |
|---|---|
| `email` | Mostrar usuario actual. |
| `userType` | Diferenciar vistas platform vs tenant. |
| `role` | Mostrar u ocultar opciones. |
| `tenantId` | Referencia de contexto, no debe enviarse manualmente en body. |

## Pendientes de decisión

- Definir almacenamiento: memory, sessionStorage, localStorage o cookie httpOnly.
- Definir expiración y logout automático.
- Definir guardas de rutas por `userType` y `role`.
- Definir manejo centralizado de errores `401` y `403`.

## Nota relacionada

- [[ADR-001 - Autenticación JWT y API Multi-Tenant]]
- [[ADR-002 - Manejo de Access Token y Refresh Token]]
- [[ADR-003 - Seguridad Avanzada de JWT y Refresh Tokens]]
