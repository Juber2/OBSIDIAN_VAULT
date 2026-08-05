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

# Authentication Authorization Claims

## Authentication

Responde: ¿quien eres? Valida credenciales o token y crea un ClaimsPrincipal.

## Authorization

Responde: ¿puedes hacer esto? Evalua roles, policies, claims, tenant y recurso.

## Claims

Claims son datos de identidad/autorizacion dentro del token: `sub`, `email`, `role`, `userType`, `tenantId`.

## Regla multi-tenant

El tenant debe venir del JWT o contexto confiable, no del body. La ruta identifica recurso; el token valida acceso.

## Errores comunes

- Mezclar login con permisos de recurso
- Creer que role basta para todo
- Aceptar tenantId del cliente

## Practica

- [ ] Definir claims para platform y tenant

## Relacionado

- [[Seguridad Auth JWT dotNET]]
- [[NET - Roadmap]]
