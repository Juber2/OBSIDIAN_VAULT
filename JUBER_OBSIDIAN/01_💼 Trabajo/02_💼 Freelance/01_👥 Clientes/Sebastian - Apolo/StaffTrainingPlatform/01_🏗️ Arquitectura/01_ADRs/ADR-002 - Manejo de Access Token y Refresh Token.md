---
tags:
  - adr
  - jwt
  - refresh-token
  - autenticacion
  - seguridad
Estado: Aceptada
Fecha: 2026-05-23
---

# ADR-002 - Manejo de Access Token y Refresh Token

## Estado

Aceptada.

## Contexto

StaffTrainingPlatform necesita manejar sesiones seguras entre frontend y backend usando JWT.

El access token debe viajar en cada request y tener una vida corta. El refresh token debe permitir renovar sesión sin obligar al usuario a iniciar sesión constantemente, pero debe poder revocarse antes de vencer.

## Decisión

Se usará una estrategia de **Access Token + Refresh Token**.

| Token | Vida | Uso |
|---|---|---|
| Access Token | 15 minutos | Viaja en cada request protegido. |
| Refresh Token | 7 días | Solo se usa para renovar el access token. |

## Regla principal del cliente

El cliente **no valida el `exp` del JWT para anticiparse**.

El cliente siempre reacciona al `401`:

1. Hace el request normal con el access token.
2. Si la API devuelve `401`, intenta refrescar.
3. Reintenta el request original con el nuevo access token.
4. Si el refresh también falla, borra tokens y redirige al login.

Este flujo debe vivir en un interceptor HTTP global:

- Angular: `HttpInterceptor`.
- React: wrapper de `axios` o `fetch`.

## Flujo completo

### Login

```text
Login -> AccessToken (15min) + RefreshToken (7 dias)
```

### Request normal

```text
Cliente -> API: request con AccessToken
API -> Cliente: 200 OK
```

### Access token vencido

```text
Cliente -> API: request con AccessToken vencido
API -> Cliente: 401 TOKEN_EXPIRED
Cliente -> API: POST /auth/refresh-token
API -> Cliente: nuevo AccessToken
Cliente -> API: reintenta request original
API -> Cliente: 200 OK
```

### Refresh token vencido

```text
Cliente -> API: POST /auth/refresh-token
API -> Cliente: 401 SESSION_EXPIRED
Cliente: borra tokens -> redirige al login
```

## Endpoint de refresh

```http
POST /auth/refresh-token
```

> Nota: si se mantiene versionado global, puede exponerse como `POST /api/v1/auth/refresh-token`.

## Códigos de error diferenciados

El body del `401` debe enviar un código específico:

```json
{
  "success": false,
  "errors": ["TOKEN_EXPIRED"]
}
```

| Código | Significado | Acción cliente |
|---|---|---|
| `TOKEN_EXPIRED` | Access token vencido. | Intentar refresh. |
| `TOKEN_INVALID` | Token manipulado o inválido. | Ir al login. |
| `SESSION_EXPIRED` | Refresh token vencido. | Ir al login. |
| `SESSION_REVOKED` | Logout, cambio de contraseña, bloqueo o revocación. | Ir al login. |

## Revocación del refresh token

El servidor invalida el refresh token cuando:

- El usuario hace logout explícito.
- El usuario cambia su contraseña.
- Un admin bloquea la cuenta.
- Se detecta actividad sospechosa.

Por eso el refresh token debe guardarse en base de datos: para poder revocarlo antes de que venza.

## Responsabilidades

| Responsable | Qué hace |
|---|---|
| API | Valida tokens, responde `401` con código específico y expone `/auth/refresh-token`. |
| Cliente | Intercepta `401`, decide si refrescar, reintenta el request original o redirige al login. |
| Base de datos | Persiste refresh tokens para permitir revocación. |

## Regla de responsabilidad

La API nunca llama sola al refresh.

La API solo responde cuando el cliente invoca el endpoint de refresh.

La única excepción son escenarios con WebSockets o SignalR, donde el servidor puede avisar proactivamente al cliente.

## Consecuencias

- El frontend no necesita leer ni validar `exp` manualmente.
- El manejo de sesión se centraliza en un interceptor.
- La API mantiene control sobre revocación mediante refresh tokens persistidos.
- Se puede diferenciar entre token vencido, token inválido, sesión vencida y sesión revocada.

## Notas relacionadas

- [[ADR-001 - Autenticación JWT y API Multi-Tenant]]
- [[Backend - Autenticación JWT]]
- [[Frontend - Manejo de JWT]]
