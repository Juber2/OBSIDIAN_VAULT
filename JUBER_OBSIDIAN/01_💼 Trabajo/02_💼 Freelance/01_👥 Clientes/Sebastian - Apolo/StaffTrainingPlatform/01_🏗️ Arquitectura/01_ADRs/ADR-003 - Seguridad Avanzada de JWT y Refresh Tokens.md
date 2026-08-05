---
tags:
  - adr
  - jwt
  - refresh-token
  - seguridad
  - token-rotation
Estado: Aceptada
Fecha: 2026-05-23
---

# ADR-003 - Seguridad Avanzada de JWT y Refresh Tokens

## Estado

Aceptada.

## Contexto

StaffTrainingPlatform usará autenticación moderna con dos tokens:

- **Access token**: viaja en cada request mediante `Authorization: Bearer`, dura 15 minutos y no requiere consultar base de datos en cada request.
- **Refresh token**: dura 7 días, se guarda en base de datos y sirve solo para renovar el access token.

El access token corto reduce el impacto si es robado. El refresh token permite mantener la sesión activa sin pedir usuario y contraseña constantemente, pero requiere mecanismos de revocación y detección de abuso.

## Por qué existen dos tokens

Usar un solo access token de larga duración, por ejemplo de 7 días, aumenta mucho el riesgo: si alguien lo intercepta, tendría acceso libre durante toda su vigencia.

Técnicamente se podría revocar consultando base de datos en cada request, pero eso elimina el beneficio principal del JWT: ser stateless y no requerir consulta a DB para cada operación.

La solución elegida es:

- Access token corto para requests frecuentes.
- Refresh token largo y revocable para renovar sesión ocasionalmente.

## Sesión potencialmente infinita

Si el usuario usa la aplicación todos los días, cada vez que el access token expira el cliente pide un nuevo par de tokens al endpoint de refresh.

Cada refresh genera:

- Nuevo access token.
- Nuevo refresh token con 7 días nuevos.

Mientras el usuario no deje pasar más de 7 días sin usar la app, la sesión puede no vencer.

Esto es intencional para evitar re-logins constantes.

## Re-login forzado

Si el negocio requiere forzar re-login cada cierto tiempo, se debe guardar la fecha del login original en el refresh token y validar un límite máximo independiente de las renovaciones.

Ejemplo:

```text
OriginalLoginAt + 30 días < Now => Forzar login
```

## Problema real del refresh token

El refresh token es una credencial de larga duración.

Si alguien lo roba y el sistema no lo detecta, puede obtener nuevos access tokens durante 7 días. En ese caso, el access token de 15 minutos deja de ser suficiente porque el atacante puede renovarlo libremente.

## Decisión principal

Se implementará **Refresh Token Rotation con detección de reuso**.

Cada vez que un refresh token se usa correctamente:

1. Se genera un nuevo access token.
2. Se genera un nuevo refresh token.
3. El refresh token anterior se marca como consumido/reemplazado.
4. El campo `ReplacedByToken` guarda el nuevo token que lo reemplazó.

El token anterior no se marca simplemente como revocado, porque fue usado en un flujo válido. Se marca como reemplazado.

## Detección de reuso

Si alguien intenta usar un refresh token que ya fue consumido/reemplazado, el sistema interpreta que hay dos actores usando la misma cadena de tokens:

- Usuario legítimo que ya renovó.
- Atacante que intenta usar un token viejo.

Acción:

- Revocar todos los tokens activos del usuario.
- Forzar re-login.
- Registrar evento de seguridad.

## Importancia de ReplacedByToken

`ReplacedByToken` permite diferenciar:

| Estado | Significado |
|---|---|
| Consumido/reemplazado | Token usado correctamente y reemplazado por otro. |
| Revocado | Token invalidado por logout, bloqueo, cambio de contraseña o seguridad. |

Sin `ReplacedByToken`, sería más difícil distinguir entre un flujo normal de renovación y una revocación por seguridad.

## Caso que la rotation no cubre

La rotation no detecta el caso en que:

1. El usuario hace login.
2. Un atacante roba el refresh token.
3. El usuario legítimo nunca vuelve a usar la app.
4. El atacante usa el refresh token primero.

Como no hubo renovación previa del usuario legítimo, no existe reuso detectable.

Para cubrir este escenario se necesita comparar el contexto de uso del refresh token contra el contexto del login original.

## Detección por contexto de login

Al crear el refresh token se guardará contexto del request:

- IP.
- País derivado de la IP.
- User-Agent.
- Navegador y sistema operativo si se parsea el User-Agent.
- Fecha del login original.

Cuando se use el refresh token, se comparará el contexto actual contra el contexto guardado.

Se considera sospechoso cuando:

- El país es diferente.
- El dispositivo es radicalmente distinto.
- Hay señales de uso anómalo.
- Hay geolocalización imposible.
- Hay demasiados intentos en muy poco tiempo.

## Flujo de notificación al usuario

Cuando se detecta uso sospechoso del refresh token, el sistema no necesariamente revoca de inmediato.

Flujo recomendado:

1. Marcar el refresh token como `pending_approval`.
2. Enviar email al usuario registrado.
3. Informar desde qué lugar/dispositivo se intenta usar la sesión.
4. Dar dos acciones: aprobar o revocar.
5. Si aprueba, se permite la sesión y se actualiza el contexto base.
6. Si revoca, se eliminan todos los tokens activos.
7. Si no responde dentro del tiempo definido, se revoca por precaución.

## Campos necesarios en RefreshToken

La entidad `RefreshToken` debe contemplar:

| Campo | Uso |
|---|---|
| `Token` | Valor del refresh token o hash del token. |
| `ExpiresAt` | Fecha de expiración. |
| `CreatedAt` | Fecha de creación. |
| `CreatedByIp` | IP original. |
| `CreatedCountry` | País del login original. |
| `UserAgent` | User-Agent original. |
| `OriginalLoginAt` | Fecha del login inicial. |
| `RevokedAt` | Fecha de revocación. |
| `RevokedReason` | Motivo de revocación. |
| `ReplacedByToken` | Token que reemplazó al actual. |
| `Status` | `active`, `pending_approval` o `revoked`. |
| `ApprovalToken` | Token único para links de aprobación/revocación por email. |

## Dónde guardar el email de notificaciones

El email de notificaciones debe vivir en la entidad `User`, no en `Person`.

Razón:

- `Person` contiene datos personales como nombre, DNI, teléfono y dirección.
- `User` contiene datos de acceso y comunicación del sistema: username, password y email.

El email es responsabilidad de la cuenta de acceso, no de la persona física.

## Otras señales de uso sospechoso

### Geolocalización imposible

Si el login fue en Lima y 5 minutos después hay un refresh desde Tokyo, es físicamente imposible.

Acción recomendada:

- Revocar automáticamente sin preguntar.

### Rate limiting

Si se hacen demasiados requests con el mismo token en poco tiempo, se considera ataque automatizado.

Endpoints mínimos con rate limiting:

- Login.
- Refresh token.

### Sesiones concurrentes desde lugares distintos

Si hay una sesión activa en Lima y aparece otra desde Madrid al mismo tiempo, se puede:

- Notificar al usuario.
- Bloquear la nueva sesión.
- Requerir aprobación.

### Horario inusual

Para empleados con horario fijo, un acceso a las 3am puede ser una señal de alerta.

Esta regla debe calibrarse para evitar falsos positivos.

## Nivel de seguridad elegido

El nivel de seguridad se calibra según el valor del activo protegido.

Para una plataforma de capacitación de staff empresarial, se considera suficiente y adecuado:

### Mínimo recomendable

- Refresh token rotation.
- Detección de reuso.
- Rate limiting en login.
- Rate limiting en refresh.

### Recomendado para este proyecto

- Refresh token rotation con detección de reuso.
- Persistencia de refresh tokens en DB.
- Device fingerprint básico.
- Límite de sesiones concurrentes.
- Notificación por email en login o refresh desde dispositivo o país nuevo.
- Detección de geolocalización imposible.

## Decisiones explícitas

- No se consultará DB en cada request solo para validar revocación del access token.
- El access token seguirá siendo corto y stateless.
- El refresh token se guardará en DB para permitir revocación anticipada.
- El refresh token rotará en cada uso exitoso.
- El reuso de refresh token reemplazado revoca todos los tokens del usuario.
- El contexto de login se usará para detectar uso sospechoso.
- El email de notificaciones pertenece a `User`.

## Notas relacionadas

- [[ADR-001 - Autenticación JWT y API Multi-Tenant]]
- [[ADR-002 - Manejo de Access Token y Refresh Token]]
- [[Backend - Autenticación JWT]]
- [[Frontend - Manejo de JWT]]
- [[Modelo base de datos - CapaActivaCorp]]
