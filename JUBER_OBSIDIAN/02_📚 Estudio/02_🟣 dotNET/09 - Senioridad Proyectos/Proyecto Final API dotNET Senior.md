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

# Proyecto Final API dotNET Senior

## Objetivo

Construir API empresarial multi-tenant para capacitación: platform admins, tenant users, cursos, inscripciones, roles y auditoria.

## Requisitos técnicos

- ASP.NET Core con controllers.
- JWT access + refresh token rotation.
- EF Core con migrations e indices.
- ProblemDetails.
- Policies por platform/tenant.
- Tests unitarios e integración.
- Dockerfile y health checks.
- OpenTelemetry o Application Insights.

## Entregables

README tecnico, ADRs de auth/arquitectura, colección HTTP, scripts de migration, tests y checklist de deploy.

## Errores comunes

- Proyecto solo CRUD
- Auth sin revocacion
- Sin tests ni observabilidad

## Practica

- [ ] Construir la API final por módulos

## Relacionado

- [[Senioridad Proyectos dotNET]]
- [[NET - Roadmap]]
