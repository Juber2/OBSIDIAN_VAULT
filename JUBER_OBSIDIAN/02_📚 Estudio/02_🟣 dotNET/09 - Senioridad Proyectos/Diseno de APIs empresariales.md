---
tags:
  - dotnet
  - Senior
  - senior
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Senioridad Proyectos dotNET
  - NET
---

# Diseno de APIs empresariales

## Contrato primero

Define recursos, status codes, errores, paginacion, filtros, auth y versionado antes de implementar.

## Multi-tenant

Contexto desde JWT, no desde body. Platform y tenant deben tener endpoints y permisos claros.

## Paginacion

Usa page/pageSize o cursor segun necesidad. Retorna metadata clara.

## Errores

ProblemDetails o formato consistente. Para auth, códigos diferenciados ayudan al frontend: TOKEN_EXPIRED, TOKEN_INVALID, SESSION_EXPIRED.

## Errores comunes

- Endpoints ambiguos
- Flags magicos en body
- Paginacion sin limites

## Practica

- [ ] Diseñar API Users platform/tenant

## Relacionado

- [[Senioridad Proyectos dotNET]]
- [[NET - Roadmap]]
