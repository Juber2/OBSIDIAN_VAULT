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

# Standalone Components

## Idea central

Un standalone component declara sus dependencias directamente en imports. Ya no necesita estar declarado en un NgModule.

## Estructura real


```ts
@Component({
  standalone: true,
  selector: 'app-user-card',
  imports: [DatePipe],
  templateUrl: './user-card.component.html',
  styleUrl: './user-card.component.scss'
})
export class UserCardComponent {
  user = input.required<User>();
}
```


## Imports explícitos

Si usas `DatePipe`, `RouterLink` o un componente hijo, debe estar en imports. Eso hace visible la dependencia.

## Providers en componente

Puedes declarar providers en un componente para crear una instancia local. Úsalo solo si necesitas scope aislado.

## Errores comunes

- Meter demasiada lógica en standalone porque parece más simple
- Importar módulos gigantes por costumbre

## Práctica

- [ ] Crear componente standalone con input requerido

## Relacionado

- [[Componentes Templates Control Flow]]
- [[Angular - Roadmap]]

## Lectura práctica

En **Standalone Components**, piensa siempre en responsabilidad del componente. Un componente sano tiene entradas claras, eventos claros y una razón concreta para existir. Si empieza a saber demasiado de HTTP, permisos, formato de DTOs o navegación, probablemente necesita apoyo de un servicio, facade o componente contenedor.

## Ejemplo de diseño

Una pantalla de usuarios puede dividirse así:

- UsersPage: carga datos, maneja estado y coordina acciones.
- UserFiltersComponent: emite filtros.
- UserCardComponent: muestra un usuario y emite selección.
- EmptyStateComponent: muestra estado vacío reutilizable.

## Criterio senior

No dividas por dividir. Divide cuando reduces cambio futuro, haces más testeable la pieza o separas una responsabilidad real. Un componente de 80 líneas puede estar bien; uno de 400 líneas con HTTP, formularios, permisos y lógica de tabla probablemente no.
