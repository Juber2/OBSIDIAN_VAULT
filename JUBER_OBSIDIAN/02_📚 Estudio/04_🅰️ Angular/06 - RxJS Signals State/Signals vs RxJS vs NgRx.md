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

# Signals vs RxJS vs NgRx

## Regla mental

Signals = estado actual. RxJS = eventos en el tiempo. NgRx = estado global complejo con convenciones fuertes.

## Cuándo Signals

Estado local, derivado, flags de UI, stores simples de feature.

## Cuándo RxJS

HTTP, formularios, router, websockets, debounce, cancelación, combinación de flujos.

## Cuándo NgRx

Muchos eventos, estado compartido por muchas features, debugging de acciones, equipo grande.

## Interop


```ts
const user = toSignal(this.api.getUser(id), { initialValue: null });
const user$ = toObservable(userSignal);
```


## Errores comunes

- Elegir NgRx por moda
- Reemplazar RxJS con signals en HTTP complejo
- Duplicar estado server y client sin invalidación

## Práctica

- [ ] Tomar una feature y justificar estrategia de estado

## Relacionado

- [[RxJS Signals State Angular]]
- [[Angular - Roadmap]]

## Modelo mental

En **Signals vs RxJS vs NgRx**, la pregunta central es: ¿estoy modelando estado actual o eventos en el tiempo? Signals brillan para estado actual y derivado. RxJS brilla para flujos async: HTTP, formularios, router, websockets, debounce, cancelación y combinación de eventos.

## Regla práctica

- Usa Signals para selectedUser, isOpen, filters y computedTotal.
- Usa RxJS para valueChanges, requests HTTP, retry, polling, websockets y cancelación.
- Usa NgRx cuando el estado global y los eventos del dominio justifican ceremonia.

## Ejemplo de arquitectura

Un store de feature puede exponer signals para la UI, pero internamente usar RxJS para cargar datos. Esa mezcla es válida si cada herramienta resuelve su problema natural. El error es forzar todo a una sola tecnología por moda.
