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

# Testing Ecosistema Senior Angular

## Objetivo

Cerrar con testing, ecosistema, escalabilidad y proyecto final.

## Objetivos de aprendizaje

- [ ] Definir estrategia de testing
- [ ] Usar Material/CDK con criterio
- [ ] Evaluar Nx/microfrontends
- [ ] Hacer code review senior

## Temas

| # | Tema | Estado | Nota |
|---|---|---|---|
| 1 | Testing Strategy Angular | ⬜ | [[Testing Strategy Angular]] |
| 2 | Angular Material CDK Design System | ⬜ | [[Angular Material CDK Design System]] |
| 3 | Nx Monorepos Microfrontends | ⬜ | [[Nx Monorepos Microfrontends]] |
| 4 | Code Review Senior Angular | ⬜ | [[Code Review Senior Angular]] |
| 5 | Proyecto Angular Senior Final | ⬜ | [[Proyecto Angular Senior Final]] |

## Práctica del módulo

- [ ] Testear componente y servicio HTTP
- [ ] Diseñar design system pequeño
- [ ] Responder preguntas senior

## Errores comunes del módulo

- E2E para todo
- Microfrontends sin necesidad
- Reviews de estilo sin riesgos reales

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

## Profundización

En **Testing Ecosistema Senior Angular**, piensa en riesgo. No todas las líneas merecen test, pero todo comportamiento crítico debería estar protegido: login, permisos, formularios, errores HTTP, mappers y navegación principal.

## Estrategia recomendada

- Unit tests para lógica pura, pipes, mappers y validators.
- Component tests para inputs, outputs y DOM visible.
- Integration tests para servicios HTTP, guards e interceptors.
- E2E para flujos críticos: login, CRUD principal y navegación.

## Criterio senior

Un test bueno falla por una regresión real, no por refactor interno. Evita probar detalles privados. Prefiere verificar comportamiento observable: texto visible, evento emitido, request hecha, ruta bloqueada o mensaje de error mostrado.
