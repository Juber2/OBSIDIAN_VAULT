---
tags:
  - Angular
  - Básico
  - Base
Creado: 2026-05-29
Nivel: Básico
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Requisitos previos Angular
  - ANGULAR
---

# HTML CSS DOM y navegador

## HTML semántico

Angular renderiza HTML. Si el HTML base es malo, la app será difícil de usar aunque Angular esté bien. Usa etiquetas semánticas para estructura, formularios y accesibilidad.

## DOM

El DOM es la representación del documento en memoria. Angular actualiza el DOM por ti, pero debes entender que templates terminan siendo nodos reales.

## CSS y layout

Angular no arregla CSS desordenado. Aprende flexbox, grid, responsive design y encapsulación de estilos.

## Ejemplo semántico


```html
<main>
  <h1>Usuarios</h1>
  <section aria-label="Listado de usuarios">
    <app-user-list />
  </section>
</main>
```


## Errores comunes

- Usar divs para todo
- Ignorar labels en formularios
- Resolver layout con hacks

## Práctica

- [ ] Crear una pantalla simple con HTML semántico y responsive CSS

## Relacionado

- [[Requisitos previos Angular]]
- [[Angular - Roadmap]]

## Por qué importa antes de Angular

En **HTML CSS DOM y navegador**, el objetivo es evitar que Angular cargue con problemas que realmente vienen de bases web. Si no entiendes HTML, HTTP, JavaScript async o CSS, terminarás culpando al framework por errores de contrato, layout o asincronía.

## Señales de dominio

- Puedes explicar qué pasa entre click, request, response y render.
- Distingues error de red, 401, 403, 404 y 500.
- Sabes cuándo un problema es CSS, template, estado o API.
- Puedes leer JSON y mapearlo a un modelo de UI.

## Mini práctica

Antes de resolverlo en Angular, intenta hacer una versión mínima con HTML, TypeScript o fetch. Si entiendes la versión simple, Angular se vuelve una herramienta; si no, Angular se vuelve una caja negra.
