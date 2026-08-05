---
tags:
  - Angular
  - Avanzado
Creado: 2026-05-29
Nivel: Avanzado
Estado: Pendiente
Relacionado:
  - ANGULAR
---

# RxJS Signals State Angular

## Objetivo

Dominar asincronía, signals y estado.

## Objetivos de aprendizaje

- [ ] Usar operadores RxJS correctos
- [ ] Usar signals/computed/effect
- [ ] Elegir Signals vs RxJS
- [ ] Diseñar estado local/feature/global

## Temas

| # | Tema | Estado | Nota |
|---|---|---|---|
| 1 | RxJS operadores clave | ⬜ | [[RxJS operadores clave]] |
| 2 | Signals computed effect | ⬜ | [[Signals computed effect]] |
| 3 | Signals vs RxJS vs NgRx | ⬜ | [[Signals vs RxJS vs NgRx]] |

## Práctica del módulo

- [ ] Buscador con switchMap
- [ ] Store de feature con signals
- [ ] Comparar NgRx vs servicio

## Errores comunes del módulo

- BehaviorSubject para todo
- effect escribiendo estado en bucle
- No cancelar requests obsoletos

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

## Modelo mental

En **RxJS Signals State Angular**, la pregunta central es: ¿estoy modelando estado actual o eventos en el tiempo? Signals brillan para estado actual y derivado. RxJS brilla para flujos async: HTTP, formularios, router, websockets, debounce, cancelación y combinación de eventos.

## Regla práctica

- Usa Signals para selectedUser, isOpen, filters y computedTotal.
- Usa RxJS para valueChanges, requests HTTP, retry, polling, websockets y cancelación.
- Usa NgRx cuando el estado global y los eventos del dominio justifican ceremonia.

## Ejemplo de arquitectura

Un store de feature puede exponer signals para la UI, pero internamente usar RxJS para cargar datos. Esa mezcla es válida si cada herramienta resuelve su problema natural. El error es forzar todo a una sola tecnología por moda.
