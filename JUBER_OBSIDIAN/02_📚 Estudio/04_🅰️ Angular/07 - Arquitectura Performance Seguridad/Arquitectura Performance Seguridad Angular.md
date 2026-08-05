---
tags:
  - Angular
  - Senior
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Relacionado:
  - ANGULAR
---

# Arquitectura Performance Seguridad Angular

## Objetivo

Diseñar apps escalables, rápidas y seguras.

## Objetivos de aprendizaje

- [ ] Aplicar feature architecture
- [ ] Usar @defer/lazy loading
- [ ] Entender SSR/hydration
- [ ] Diseñar auth frontend segura

## Temas

| # | Tema | Estado | Nota |
|---|---|---|---|
| 1 | Feature Architecture Boundaries | ⬜ | [[Feature Architecture Boundaries]] |
| 2 | Facade Repository Adapter | ⬜ | [[Facade Repository Adapter]] |
| 3 | Performance @defer OnPush Zoneless | ⬜ | [[Performance @defer OnPush Zoneless]] |
| 4 | SSR Hydration y Rendering | ⬜ | [[SSR Hydration y Rendering]] |
| 5 | Auth JWT Refresh Tokens RBAC | ⬜ | [[Auth JWT Refresh Tokens RBAC]] |

## Práctica del módulo

- [ ] Diseñar feature users
- [ ] Aplicar @defer a chart
- [ ] Crear guard + interceptor auth

## Errores comunes del módulo

- Shared gigante
- SSR sin necesidad
- Frontend como autoridad de permisos

## Recursos oficiales

- Angular Docs: https://angular.dev/
- Components: https://angular.dev/guide/components
- Templates: https://angular.dev/guide/templates
- Signals: https://angular.dev/guide/signals
- Router: https://angular.dev/guide/routing
- Forms: https://angular.dev/guide/forms
- HTTP: https://angular.dev/guide/http
- Defer: https://angular.dev/guide/templates/defer
- Hydration: https://angular.dev/guide/hydration
- Testing: https://angular.dev/guide/testing

## Decisión técnica

En **Arquitectura Performance Seguridad Angular**, no busques una receta única. Angular senior consiste en elegir estructura según tamaño del equipo, vida útil del producto, cantidad de features, necesidad de SEO, performance, permisos y testing.

## Preguntas antes de decidir

- ¿Este código cambiará por negocio o por infraestructura?
- ¿La feature tiene ownership claro?
- ¿Necesita estado compartido o solo local?
- ¿Hay rutas lazy y boundaries?
- ¿Qué parte debe poder testearse sin levantar toda la app?

## Criterio de madurez

Una arquitectura buena no se ve solo en carpetas bonitas. Se ve cuando agregar una feature no rompe otra, cuando una PR es revisable, cuando los tests cubren riesgos y cuando el equipo entiende dónde poner cada pieza.
