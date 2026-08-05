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

# ControlValueAccessor y formularios reutilizables

## Cuándo usar CVA

Cuando un componente custom debe funcionar con `formControlName` como si fuera input nativo.

## Métodos clave

- `writeValue` recibe valor externo.
- `registerOnChange` registra callback para cambios.
- `registerOnTouched` registra touch.
- `setDisabledState` aplica disabled.

## Ejemplo conceptual


```ts
export class MoneyInputComponent implements ControlValueAccessor {
  private onChange = (value: number) => {};

  writeValue(value: number): void {}
  registerOnChange(fn: (value: number) => void): void { this.onChange = fn; }
  registerOnTouched(fn: () => void): void {}
}
```


## Criterio

No crees CVA para inputs simples. Úsalo para componentes con UI/comportamiento propio: money input, autocomplete, uploader, date range.

## Errores comunes

- CVA que no propaga touched
- No soportar disabled
- Mezclar formato visual con valor real

## Práctica

- [ ] Crear componente RatingInput con CVA

## Relacionado

- [[Routing Forms UX Angular]]
- [[Angular - Roadmap]]

## Aplicación en una app real

En **ControlValueAccessor y formularios reutilizables**, lo importante es diseñar flujos de usuario, no solo configurar APIs. Routing, formularios y UX se cruzan: una mala ruta complica permisos; un mal formulario genera datos inválidos; un mal estado visual hace que el usuario repita acciones.

## Flujo recomendado

1. Define rutas por intención: listar, crear, editar, detalle.
2. Usa params para identidad de recurso y query params para filtros.
3. Modela estados del formulario: pristine, dirty, invalid, pending, saving, saved, failed.
4. Maneja navegación después de guardar sin perder feedback.

## Criterio senior

Un formulario grande no debe ser un bloque enorme de HTML y TypeScript. Divide secciones, crea componentes reutilizables cuando haya comportamiento real, y centraliza mensajes de error si se repiten demasiado.
