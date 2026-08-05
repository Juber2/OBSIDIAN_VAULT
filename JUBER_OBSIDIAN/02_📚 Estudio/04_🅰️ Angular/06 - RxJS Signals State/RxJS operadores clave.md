---
tags:
  - Angular
  - Avanzado
  - Reactividad
Creado: 2026-05-29
Nivel: Avanzado
Estado: Pendiente
Prioridad: Media
Relacionado:
  - RxJS Signals State Angular
  - ANGULAR
---

# RxJS operadores clave

## Observable

Secuencia de valores en el tiempo. HTTP, formularios y router usan Observables.

## Flattening operators

| Operador | Estrategia | Caso típico |
|---|---|---|
| switchMap | cancela anterior | búsqueda |
| mergeMap | paralelo | acciones independientes |
| concatMap | cola | guardados ordenados |
| exhaustMap | ignora nuevos | login/submit |

## Ejemplo búsqueda


```ts
results$ = this.search.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.api.search(term)),
  catchError(() => of([]))
);
```


## shareReplay

Útil para compartir resultado. Peligroso si cacheas datos que deben invalidarse.

## Errores comunes

- Nested subscribe
- No manejar errores
- Usar mergeMap donde necesitas cancelar

## Práctica

- [ ] Crear buscador con debounce y switchMap

## Relacionado

- [[RxJS Signals State Angular]]
- [[Angular - Roadmap]]

## Modelo mental

En **RxJS operadores clave**, la pregunta central es: ¿estoy modelando estado actual o eventos en el tiempo? Signals brillan para estado actual y derivado. RxJS brilla para flujos async: HTTP, formularios, router, websockets, debounce, cancelación y combinación de eventos.

## Regla práctica

- Usa Signals para selectedUser, isOpen, filters y computedTotal.
- Usa RxJS para valueChanges, requests HTTP, retry, polling, websockets y cancelación.
- Usa NgRx cuando el estado global y los eventos del dominio justifican ceremonia.

## Ejemplo de arquitectura

Un store de feature puede exponer signals para la UI, pero internamente usar RxJS para cargar datos. Esa mezcla es válida si cada herramienta resuelve su problema natural. El error es forzar todo a una sola tecnología por moda.
