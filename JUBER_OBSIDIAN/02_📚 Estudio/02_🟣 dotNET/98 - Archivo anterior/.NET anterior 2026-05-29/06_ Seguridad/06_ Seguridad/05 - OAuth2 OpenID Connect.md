---
tags:
  - NET
  - OAuth2
  - OIDC
Creado: 2026-05-21
Relacionado:
  - Seguridad en .NET
---

# OAuth2 y OpenID Connect

OAuth2 delega autorización. OpenID Connect agrega identidad encima de OAuth2.

## Conceptos

| Concepto | Significado |
|---|---|
| Authorization Server | emite tokens |
| Resource Server | API protegida |
| Client | aplicación que solicita acceso |
| Access Token | acceso a recursos |
| ID Token | identidad del usuario |

## Flujos comunes

- Authorization Code + PKCE para frontend/mobile.
- Client Credentials para comunicación server-to-server.

## Checklist

- [ ] Sé cuándo usar OAuth2/OIDC.
- [ ] No uso implicit flow en apps modernas.
- [ ] Valido issuer, audience y firma.
- [ ] Entiendo access token vs id token.
