---
tags:
  - Angular
  - Básico
Creado: 2026-05-29
Nivel: Básico
Estado: Pendiente
Relacionado:
  - ANGULAR
---

# Componentes Templates Control Flow

## Objetivo

Construir UI moderna con standalone components y templates claros.

## Objetivos de aprendizaje

- [ ] Crear standalone components
- [ ] Usar input/output/model
- [ ] Dominar binding
- [ ] Usar @if/@for/@switch con track

## Temas

| # | Tema | Estado | Nota |
|---|---|---|---|
| 1 | Standalone Components | ⬜ | [[Standalone Components]] |
| 2 | Inputs Outputs y model input | ⬜ | [[Inputs Outputs y model input]] |
| 3 | Templates Binding y Async Pipe | ⬜ | [[Templates Binding y Async Pipe]] |
| 4 | Control Flow moderno | ⬜ | [[Control Flow moderno]] |

## Práctica del módulo

- [ ] Crear UserCard presentacional
- [ ] Crear UsersPage contenedor
- [ ] Migrar lista a @for con track

## Errores comunes del módulo

- Componentes gigantes
- Templates con lógica pesada
- No usar track en listas

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

## Lectura práctica

En **Componentes Templates Control Flow**, piensa siempre en responsabilidad del componente. Un componente sano tiene entradas claras, eventos claros y una razón concreta para existir. Si empieza a saber demasiado de HTTP, permisos, formato de DTOs o navegación, probablemente necesita apoyo de un servicio, facade o componente contenedor.

## Ejemplo de diseño

Una pantalla de usuarios puede dividirse así:

- UsersPage: carga datos, maneja estado y coordina acciones.
- UserFiltersComponent: emite filtros.
- UserCardComponent: muestra un usuario y emite selección.
- EmptyStateComponent: muestra estado vacío reutilizable.

## Criterio senior

No dividas por dividir. Divide cuando reduces cambio futuro, haces más testeable la pieza o separas una responsabilidad real. Un componente de 80 líneas puede estar bien; uno de 400 líneas con HTTP, formularios, permisos y lógica de tabla probablemente no.
