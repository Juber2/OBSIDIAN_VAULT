---
tags:
  - dotnet
  - Intermedio
  - seguridad
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Seguridad Auth JWT dotNET
  - NET
---

# Refresh Tokens rotation y reuso

## Refresh token

Credencial larga que solo sirve para renovar access token. Debe persistirse en base de datos para revocacion.

## Rotation

Cada refresh valido genera nuevo refresh token y marca el anterior como reemplazado. Si alguien usa un token ya consumido, se detecta posible robo.

## Campos utiles

`TokenHash`, `UserId`, `ExpiresAt`, `RevokedAt`, `ReplacedByToken`, `Status`, `IpAddress`, `UserAgent`, `Country`, `CreatedAt`.

## Accion ante reuso

Revocar toda la cadena de tokens del usuario y forzar login.

## Errores comunes

- Guardar refresh token en texto plano
- No detectar reuso
- No invalidar tokens al cambiar contraseña

## Practica

- [ ] Modelar entidad RefreshToken con rotation

## Relacionado

- [[Seguridad Auth JWT dotNET]]
- [[NET - Roadmap]]
