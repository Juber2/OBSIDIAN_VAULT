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

# Typed Reactive Forms

## FormControl FormGroup FormArray

Reactive Forms modela estructura, valor, estado y validación desde TypeScript.

## Typed forms

Permiten que `form.value` y controles tengan tipos seguros.

## Ejemplo


```ts
type UserForm = {
  email: FormControl<string>;
  role: FormControl<'admin' | 'employee'>;
};

form = new FormGroup<UserForm>({
  email: new FormControl('', { nonNullable: true, validators: [Validators.required, Validators.email] }),
  role: new FormControl('employee', { nonNullable: true })
});
```


## Errores de UI

Muestra errores cuando el campo fue tocado o después de submit. No castigues al usuario antes de interactuar.

## Validación backend

La validación frontend mejora UX; la validación backend protege datos. Necesitas ambas.

## Errores comunes

- Formularios sin tipos
- Mensajes de error duplicados en cada template
- Validadores async sin debounce

## Práctica

- [ ] Crear formulario typed de registro

## Relacionado

- [[Routing Forms UX Angular]]
- [[Angular - Roadmap]]

## Aplicación en una app real

En **Typed Reactive Forms**, lo importante es diseñar flujos de usuario, no solo configurar APIs. Routing, formularios y UX se cruzan: una mala ruta complica permisos; un mal formulario genera datos inválidos; un mal estado visual hace que el usuario repita acciones.

## Flujo recomendado

1. Define rutas por intención: listar, crear, editar, detalle.
2. Usa params para identidad de recurso y query params para filtros.
3. Modela estados del formulario: pristine, dirty, invalid, pending, saving, saved, failed.
4. Maneja navegación después de guardar sin perder feedback.

## Criterio senior

Un formulario grande no debe ser un bloque enorme de HTML y TypeScript. Divide secciones, crea componentes reutilizables cuando haya comportamiento real, y centraliza mensajes de error si se repiten demasiado.
