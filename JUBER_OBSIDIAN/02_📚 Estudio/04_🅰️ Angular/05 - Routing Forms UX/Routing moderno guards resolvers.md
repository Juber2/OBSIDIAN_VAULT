---
tags:
  - Angular
  - Intermedio
  - Routing
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Routing Forms UX Angular
  - ANGULAR
---

# Routing moderno guards resolvers

## Routes modernas

Puedes cargar componentes standalone con `loadComponent` y features con `loadChildren`.

## Ejemplo ruta lazy


```ts
export const routes: Routes = [
  {
    path: 'users',
    loadComponent: () => import('./features/users/users.page')
      .then(m => m.UsersPage)
  }
];
```


## Params y query params

Path params identifican recursos: `/users/:id`. Query params expresan filtros/paginación: `?page=2&search=ana`.

## Guards

Controlan navegación, pero no reemplazan validación backend.

## Resolvers

Pueden precargar datos, pero si bloquean demasiado la navegación quizá sea mejor mostrar skeleton.

## Errores comunes

- Usar guards como seguridad real única
- Meter carga lenta en resolver sin UX
- Rutas gigantes en un solo archivo

## Práctica

- [ ] Crear ruta lazy con guard funcional

## Relacionado

- [[Routing Forms UX Angular]]
- [[Angular - Roadmap]]

## Aplicación en una app real

En **Routing moderno guards resolvers**, lo importante es diseñar flujos de usuario, no solo configurar APIs. Routing, formularios y UX se cruzan: una mala ruta complica permisos; un mal formulario genera datos inválidos; un mal estado visual hace que el usuario repita acciones.

## Flujo recomendado

1. Define rutas por intención: listar, crear, editar, detalle.
2. Usa params para identidad de recurso y query params para filtros.
3. Modela estados del formulario: pristine, dirty, invalid, pending, saving, saved, failed.
4. Maneja navegación después de guardar sin perder feedback.

## Criterio senior

Un formulario grande no debe ser un bloque enorme de HTML y TypeScript. Divide secciones, crea componentes reutilizables cuando haya comportamiento real, y centraliza mensajes de error si se repiten demasiado.
