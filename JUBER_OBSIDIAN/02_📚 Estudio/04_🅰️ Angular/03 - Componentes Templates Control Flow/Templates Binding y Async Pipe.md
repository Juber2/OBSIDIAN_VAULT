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

# Templates Binding y Async Pipe

## Interpolación

`{{ value }}` renderiza texto escapado. Úsalo para mostrar valores.

## Property binding

`[disabled]="isSaving()"` pasa valores a propiedades o inputs.

## Event binding

`(click)="save()"` reacciona a eventos.

## Async pipe

Suscribe Observables y limpia suscripción automáticamente.

## Ejemplo


```html
@if (user$ | async; as user) {
  <app-user-card [user]="user" (selected)="selectUser($event)" />
}
```


## Qué evitar

Evita llamar funciones costosas en template, crear objetos inline en loops y meter reglas de negocio en HTML.

## Errores comunes

- Lógica pesada en template
- Funciones que se recalculan todo el tiempo
- Templates enormes sin componentes hijos

## Práctica

- [ ] Crear template con loading/error/empty/success

## Relacionado

- [[Componentes Templates Control Flow]]
- [[Angular - Roadmap]]

## Lectura práctica

En **Templates Binding y Async Pipe**, piensa siempre en responsabilidad del componente. Un componente sano tiene entradas claras, eventos claros y una razón concreta para existir. Si empieza a saber demasiado de HTTP, permisos, formato de DTOs o navegación, probablemente necesita apoyo de un servicio, facade o componente contenedor.

## Ejemplo de diseño

Una pantalla de usuarios puede dividirse así:

- UsersPage: carga datos, maneja estado y coordina acciones.
- UserFiltersComponent: emite filtros.
- UserCardComponent: muestra un usuario y emite selección.
- EmptyStateComponent: muestra estado vacío reutilizable.

## Criterio senior

No dividas por dividir. Divide cuando reduces cambio futuro, haces más testeable la pieza o separas una responsabilidad real. Un componente de 80 líneas puede estar bien; uno de 400 líneas con HTTP, formularios, permisos y lógica de tabla probablemente no.
