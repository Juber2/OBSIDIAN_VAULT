---
tags:
  - Postulaciones
  - Angular
  - Entrevistas
Creado: 2026-08-30
Relacionado:
  - ANGULAR
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Ruta de repaso Angular para entrevistas

## Objetivo

Poder construir y explicar una feature vertical: ruta → página → formulario → servicio → API → estados de UI → pruebas.

## Día 1 — TypeScript y componentes

- Tipos, interfaces, unions, generics y null safety.
- Standalone components, inputs/outputs, templates y control flow.
- Ciclo de vida y composición.

Repasar: [[TypeScript esencial para Angular]], [[Standalone Components]], [[Control Flow moderno]].

**Prueba:** crear un componente reutilizable tipado sin usar `any`.

## Día 2 — DI, HTTP y arquitectura de feature

- Providers e inyección.
- HttpClient, DTOs, mapping, interceptors y errores.
- Separación entre page, UI, data-access y dominio.

Repasar: [[Dependency Injection e InjectionToken]], [[HttpClient DTOs mapping e interceptors]], [[Feature Architecture Boundaries]].

**Prueba:** implementar un flujo CRUD con loading, error y empty state.

## Día 3 — Forms y routing

- Typed Reactive Forms y validaciones.
- Guards, resolvers, lazy loading y parámetros.
- UX de errores y navegación.

Repasar: [[Typed Reactive Forms]], [[Routing moderno guards resolvers]].

**Prueba:** formulario create/edit que maneje errores del backend.

## Día 4 — RxJS, Signals y estado

- Observable, Subject, `switchMap`, `mergeMap`, `concatMap`, `exhaustMap`.
- Cancelación, suscripciones y `async` pipe.
- `signal`, `computed`, `effect` y cuándo no usar un effect.
- Estado local, servicio de estado o NgRx.

Repasar: [[RxJS operadores clave]], [[Signals computed effect]], [[Signals vs RxJS vs NgRx]].

**Prueba:** búsqueda con debounce, cancelación y estados de UI.

## Día 5 — Seguridad, performance y testing

- JWT, refresh, XSS, CORS y autoridad del backend.
- Lazy loading, `@defer`, rendering y medición.
- Unit/component tests y prueba del contrato HTTP.

Repasar: [[Auth JWT Refresh Tokens RBAC]], [[Performance @defer OnPush Zoneless]], [[Testing Strategy Angular]].

**Prueba:** proteger una ruta, probar interceptor y evitar un refresh loop.

## Preguntas que debes poder responder

- ¿Signals reemplaza RxJS?
- ¿Qué diferencia existe entre un guard y seguridad real?
- ¿Cómo evitas memory leaks y solicitudes obsoletas?
- ¿Cuándo usarías NgRx?
- ¿Cómo estructuras una feature grande?
- ¿Cómo diagnosticarías renders o bundle excesivos?

Usar [[Preguntas de Entrevista Angular]] y [[Preguntas Angular Senior Casos Reales]] para simulaciones.

## Criterio de salida

- [ ] Implemento una feature completa en menos de dos horas.
- [ ] Justifico Signals, RxJS y estado global con trade-offs.
- [ ] Pruebo los flujos críticos.
- [ ] Puedo explicar una decisión de performance y una de seguridad.

