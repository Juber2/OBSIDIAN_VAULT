---
tags:
  - angular
  - laboratorio
  - componentes
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos Angular
  - ANGULAR
---

# Laboratorio 02 - Componentes templates y control flow

## Objetivo

Construir una pantalla de listado con componentes presentacionales, inputs/outputs, control flow moderno y estados loading/empty/error/success.

## Componentes

- `UsersPage` como contenedor.
- `UserCardComponent` presentacional.
- `UserFiltersComponent`.
- `EmptyStateComponent`.

## Template esperado


```html
@switch (state()) {
  @case ('loading') { <app-spinner /> }
  @case ('error') { <app-error-state /> }
  @case ('success') {
    @for (user of users(); track user.id) {
      <app-user-card [user]="user" (selected)="selectUser($event)" />
    } @empty {
      <app-empty-state />
    }
  }
}
```


## Criterio de terminado

El template no contiene reglas de negocio pesadas y toda lista usa `track` estable.

## Errores comunes

- Componente gigante
- No modelar estados
- Usar track $index en listas reordenables

## Practica

- [ ] Crear estados de UI
- [ ] Dividir componente grande
- [ ] Revisar accesibilidad básica

## Relacionado

- [[Laboratorios Practicos Angular]]
- [[ANGULAR]]

## Desarrollo guiado

Este laboratorio no se debe resolver copiando snippets aislados. La idea es construir una pequeña pieza funcional, ejecutar la app y registrar qué decisiones tomaste. Para **Laboratorio 02 - Componentes templates y control flow**, trabaja en tres capas: primero haz que funcione, luego ordena responsabilidades y finalmente agrega validación o prueba mínima.

## Pasos sugeridos

1. Crea una rama o carpeta de práctica con nombre claro.
2. Implementa el flujo mínimo sin optimizar antes de tiempo.
3. Separa UI, estado y acceso a datos si el ejercicio lo justifica.
4. Agrega al menos un caso de error: API falla, formulario inválido, sesión expirada o lista vacía.
5. Escribe una nota corta con lo que aprendiste y lo que cambiarías.

## Criterio de revisión

El laboratorio está completo cuando otra persona puede abrir el código, entender el flujo y ejecutarlo. Revisa nombres, estados de UI, manejo de errores y que no hayas escondido reglas importantes dentro del template.
