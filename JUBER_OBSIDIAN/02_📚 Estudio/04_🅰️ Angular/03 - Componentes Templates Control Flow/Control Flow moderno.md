---
tags:
  - Angular
  - Básico
  - Componentes
Creado: 2026-05-29
Nivel: Básico
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Componentes Templates Control Flow
  - ANGULAR
---

# Control Flow moderno

## @if/@else

Condiciones legibles sin `*ngIf`.

## @for/@empty

Listas con estado vacío y tracking obligatorio.

## @switch

Sirve para renderizar según estado discreto, como `loading`, `success`, `error`.

## Ejemplo completo


```html
@switch (state()) {
  @case ('loading') { <app-spinner /> }
  @case ('error') { <app-error /> }
  @case ('success') {
    @for (user of users(); track user.id) {
      <app-user-card [user]="user" />
    } @empty {
      <p>No hay usuarios</p>
    }
  }
}
```


## track

Usa un identificador estable: `track user.id`. Evita `track $index` si la lista puede reordenarse.

## Errores comunes

- No usar track
- Anidar demasiados bloques
- Usar @if para esconder problemas de estado mal modelado

## Práctica

- [ ] Migrar una lista legacy *ngFor a @for

## Relacionado

- [[Componentes Templates Control Flow]]
- [[Angular - Roadmap]]

## Lectura práctica

En **Control Flow moderno**, piensa siempre en responsabilidad del componente. Un componente sano tiene entradas claras, eventos claros y una razón concreta para existir. Si empieza a saber demasiado de HTTP, permisos, formato de DTOs o navegación, probablemente necesita apoyo de un servicio, facade o componente contenedor.

## Ejemplo de diseño

Una pantalla de usuarios puede dividirse así:

- UsersPage: carga datos, maneja estado y coordina acciones.
- UserFiltersComponent: emite filtros.
- UserCardComponent: muestra un usuario y emite selección.
- EmptyStateComponent: muestra estado vacío reutilizable.

## Criterio senior

No dividas por dividir. Divide cuando reduces cambio futuro, haces más testeable la pieza o separas una responsabilidad real. Un componente de 80 líneas puede estar bien; uno de 400 líneas con HTTP, formularios, permisos y lógica de tabla probablemente no.
