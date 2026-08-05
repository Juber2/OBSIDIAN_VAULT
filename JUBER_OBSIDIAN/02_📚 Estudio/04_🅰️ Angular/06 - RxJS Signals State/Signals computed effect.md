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

# Signals computed effect

## signal

Estado actual. Se lee con `count()` y se escribe con `set` o `update`.

## computed

Valor derivado y puro. No hagas HTTP ni side effects dentro.

## effect

Side effect reactivo. Úsalo para logging, sincronización externa o integración, no para calcular estado derivado.

## Ejemplo


```ts
const users = signal<User[]>([]);
const activeUsers = computed(() => users().filter(user => user.active));

effect(() => {
  console.log('Activos', activeUsers().length);
});
```


## Signals en servicios

Un servicio de feature puede exponer signals readonly y métodos para mutar estado.

## Errores comunes

- Mutar arrays dentro de signal sin set/update
- effect que escribe signals en ciclo
- Usar signals para streams complejos

## Práctica

- [ ] Crear contador y lista filtrada con computed

## Relacionado

- [[RxJS Signals State Angular]]
- [[Angular - Roadmap]]

## Modelo mental

En **Signals computed effect**, la pregunta central es: ¿estoy modelando estado actual o eventos en el tiempo? Signals brillan para estado actual y derivado. RxJS brilla para flujos async: HTTP, formularios, router, websockets, debounce, cancelación y combinación de eventos.

## Regla práctica

- Usa Signals para selectedUser, isOpen, filters y computedTotal.
- Usa RxJS para valueChanges, requests HTTP, retry, polling, websockets y cancelación.
- Usa NgRx cuando el estado global y los eventos del dominio justifican ceremonia.

## Ejemplo de arquitectura

Un store de feature puede exponer signals para la UI, pero internamente usar RxJS para cargar datos. Esa mezcla es válida si cada herramienta resuelve su problema natural. El error es forzar todo a una sola tecnología por moda.
