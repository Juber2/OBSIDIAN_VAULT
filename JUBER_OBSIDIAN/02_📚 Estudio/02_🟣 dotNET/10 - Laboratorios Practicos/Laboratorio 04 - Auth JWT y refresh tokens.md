---
tags:
  - dotnet
  - laboratorio
  - seguridad
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos dotNET
  - NET
---

# Laboratorio 04 - Auth JWT y refresh tokens

## Objetivo

Implementar login, access token corto, refresh token persistido, rotation, revocacion y errores diferenciados.

## Flujo

1. Login valida credenciales.
2. API emite access token y refresh token.
3. Cliente usa access token.
4. Si expira, cliente llama refresh.
5. Refresh valido rota token.
6. Reuso de refresh token revoca la cadena.

## Entidad RefreshToken

Debe guardar hash del token, expiracion, estado, reemplazo, fecha de revocacion, user agent e IP si aplica.

## Codigos de error

`TOKEN_EXPIRED`, `TOKEN_INVALID`, `SESSION_EXPIRED`, `SESSION_REVOKED`.

## Errores comunes

- Guardar refresh token plano
- No revocar al cambiar contraseña
- No diferenciar 401 y 403

## Practica

- [ ] Crear login
- [ ] Crear refresh endpoint
- [ ] Testear reuso detectado

## Relacionado

- [[Laboratorios Practicos dotNET]]
- [[NET]]
