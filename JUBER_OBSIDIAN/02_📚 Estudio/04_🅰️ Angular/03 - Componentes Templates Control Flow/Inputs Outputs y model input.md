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

# Inputs Outputs y model input

## input()

Define entradas tipadas. `input.required<T>()` obliga a que el padre pase el valor.

## output()

Define eventos tipados que el componente emite al padre.

## model()

Sirve para two-way binding moderno en componentes de edición.

## Ejemplo


```ts
export class QuantitySelectorComponent {
  value = model(1);
  changed = output<number>();

  increment() {
    this.value.update(v => v + 1);
    this.changed.emit(this.value());
  }
}
```


## Uso


```html
<app-quantity-selector [(value)]="quantity" (changed)="save($event)" />
```


## Errores comunes

- Mutar objetos recibidos por input
- Emitir eventos ambiguos como changed sin payload claro
- Usar model para todo

## Práctica

- [ ] Crear RatingComponent con model input

## Relacionado

- [[Componentes Templates Control Flow]]
- [[Angular - Roadmap]]

## Lectura práctica

En **Inputs Outputs y model input**, piensa siempre en responsabilidad del componente. Un componente sano tiene entradas claras, eventos claros y una razón concreta para existir. Si empieza a saber demasiado de HTTP, permisos, formato de DTOs o navegación, probablemente necesita apoyo de un servicio, facade o componente contenedor.

## Ejemplo de diseño

Una pantalla de usuarios puede dividirse así:

- UsersPage: carga datos, maneja estado y coordina acciones.
- UserFiltersComponent: emite filtros.
- UserCardComponent: muestra un usuario y emite selección.
- EmptyStateComponent: muestra estado vacío reutilizable.

## Criterio senior

No dividas por dividir. Divide cuando reduces cambio futuro, haces más testeable la pieza o separas una responsabilidad real. Un componente de 80 líneas puede estar bien; uno de 400 líneas con HTTP, formularios, permisos y lógica de tabla probablemente no.
