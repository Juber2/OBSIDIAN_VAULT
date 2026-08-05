---
tags:
  - Angular
  - Intermedio
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Relacionado:
  - ANGULAR
---

# Routing Forms UX Angular

## Objetivo

Dominar navegación, guards y formularios reales.

## Objetivos de aprendizaje

- [ ] Configurar rutas lazy
- [ ] Usar params/query params
- [ ] Crear typed reactive forms
- [ ] Implementar CVA cuando corresponde

## Temas

| # | Tema | Estado | Nota |
|---|---|---|---|
| 1 | Routing moderno guards resolvers | ⬜ | [[Routing moderno guards resolvers]] |
| 2 | Typed Reactive Forms | ⬜ | [[Typed Reactive Forms]] |
| 3 | ControlValueAccessor y formularios reutilizables | ⬜ | [[ControlValueAccessor y formularios reutilizables]] |

## Práctica del módulo

- [ ] Crear rutas users lazy
- [ ] Crear guard auth
- [ ] Crear formulario con validadores

## Errores comunes del módulo

- Validación solo visual
- Rutas sin lazy loading
- Formularios enormes en un componente

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

## Aplicación en una app real

En **Routing Forms UX Angular**, lo importante es diseñar flujos de usuario, no solo configurar APIs. Routing, formularios y UX se cruzan: una mala ruta complica permisos; un mal formulario genera datos inválidos; un mal estado visual hace que el usuario repita acciones.

## Flujo recomendado

1. Define rutas por intención: listar, crear, editar, detalle.
2. Usa params para identidad de recurso y query params para filtros.
3. Modela estados del formulario: pristine, dirty, invalid, pending, saving, saved, failed.
4. Maneja navegación después de guardar sin perder feedback.

## Criterio senior

Un formulario grande no debe ser un bloque enorme de HTML y TypeScript. Divide secciones, crea componentes reutilizables cuando haya comportamiento real, y centraliza mensajes de error si se repiten demasiado.
