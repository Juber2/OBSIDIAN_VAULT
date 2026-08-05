---
tags:
  - Angular
  - Senior
  - Senior
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Arquitectura Performance Seguridad Angular
  - ANGULAR
---

# Performance @defer OnPush Zoneless

## @defer

Difiere dependencias standalone que no son necesarias en el primer render.

## Bloques

`@placeholder`, `@loading`, `@error`, triggers como `on viewport`, y `prefetch` para descargar antes.

## Ejemplo


```html
@defer (on viewport; prefetch on idle) {
  <app-heavy-chart />
} @placeholder {
  <app-chart-skeleton />
} @loading {
  <app-spinner />
}
```


## OnPush y signals

Reducen trabajo de change detection cuando el estado cambia de forma explícita.

## Zoneless

Angular puede funcionar sin ZoneJS usando APIs de notificación más explícitas.

## Errores comunes

- Optimizar sin medir
- Usar @defer en contenido crítico above the fold
- No definir placeholder/loading

## Práctica

- [ ] Medir pantalla con Angular DevTools y diferir un widget pesado

## Relacionado

- [[Arquitectura Performance Seguridad Angular]]
- [[Angular - Roadmap]]

## Decisión técnica

En **Performance @defer OnPush Zoneless**, no busques una receta única. Angular senior consiste en elegir estructura según tamaño del equipo, vida útil del producto, cantidad de features, necesidad de SEO, performance, permisos y testing.

## Preguntas antes de decidir

- ¿Este código cambiará por negocio o por infraestructura?
- ¿La feature tiene ownership claro?
- ¿Necesita estado compartido o solo local?
- ¿Hay rutas lazy y boundaries?
- ¿Qué parte debe poder testearse sin levantar toda la app?

## Criterio de madurez

Una arquitectura buena no se ve solo en carpetas bonitas. Se ve cuando agregar una feature no rompe otra, cuando una PR es revisable, cuando los tests cubren riesgos y cuando el equipo entiende dónde poner cada pieza.
