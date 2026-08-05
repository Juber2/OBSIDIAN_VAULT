---
tags:
  - arquitectura
  - multi-tenant
  - staff-training-platform
Actualizado: 2026-05-22
---

# Arquitectura StaffTrainingPlatform

## Contextos principales

El sistema se diseña con dos contextos explícitos:

| Contexto | Descripción |
|---|---|
| Platform | Administradores globales de la plataforma. No pertenecen a ningún tenant específico. |
| Tenant | Usuarios que operan dentro de una empresa o tenant específico. |

## Principios de diseño

- La ruta identifica el recurso.
- El JWT valida acceso y contexto.
- El tenant activo se obtiene desde claims del JWT.
- El body no debe cambiar comportamiento global del endpoint.
- Query params se reservan para filtros, paginación y ordenamiento.
- Login es el único endpoint público.

## Decisiones relacionadas

- [[ADR-001 - Autenticación JWT y API Multi-Tenant]]
- [[ADR-002 - Manejo de Access Token y Refresh Token]]
- [[ADR-003 - Seguridad Avanzada de JWT y Refresh Tokens]]
- [[Backend - API Multi-Tenant]]
- [[Backend - Autenticación JWT]]
- [[Frontend - Manejo de JWT]]

## Áreas por definir

- Modelo de autorización por policies.
- Separación de módulos en backend.
- Modelo final de persistencia de refresh tokens.
- Estrategia de device fingerprint básico.
- Manejo de sesiones y revocación.
- Auditoría por tenant.
