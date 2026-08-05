---
tags:
  - NET
  - Arquitectura
  - Proyecto
Creado: 2026-05-21
Relacionado:
  - Proyectos .NET
---

# Proyecto Modular Monolith .NET

## Objetivo

Crear una aplicación grande en un despliegue, pero organizada por módulos.

## Módulos sugeridos

- Users.
- Catalog.
- Orders.
- Payments.
- Notifications.

## Debe incluir

- Clean Architecture por módulo.
- Eventos internos.
- EF Core.
- Tests.
- ADRs.

## Checklist

- [ ] Boundaries claros.
- [ ] Sin dependencias circulares.
- [ ] SharedKernel pequeño.
- [ ] Cada módulo se entiende solo.
