---
tags:
  - angular
  - laboratorio
  - reactividad
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos Angular
  - ANGULAR
---

# Laboratorio 04 - RxJS Signals y estado

## Objetivo

Combinar RxJS para eventos/HTTP y Signals para estado actual de UI sin mezclar responsabilidades.

## Caso

Buscador de usuarios con debounce, cancelación de requests obsoletos, loading state y cache simple.

## RxJS para búsqueda


```ts
results$ = searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => api.searchUsers(term))
);
```


## Signals para estado

Usa `signal`, `computed` y métodos explícitos en un store de feature para filtros, selección y flags de UI.

## Errores comunes

- BehaviorSubject para todo
- effect que modifica signals en ciclo
- No cancelar búsquedas anteriores

## Practica

- [ ] Crear buscador con switchMap
- [ ] Crear store simple con signals
- [ ] Comparar ambos enfoques

## Relacionado

- [[Laboratorios Practicos Angular]]
- [[ANGULAR]]

## Desarrollo guiado

Este laboratorio no se debe resolver copiando snippets aislados. La idea es construir una pequeña pieza funcional, ejecutar la app y registrar qué decisiones tomaste. Para **Laboratorio 04 - RxJS Signals y estado**, trabaja en tres capas: primero haz que funcione, luego ordena responsabilidades y finalmente agrega validación o prueba mínima.

## Pasos sugeridos

1. Crea una rama o carpeta de práctica con nombre claro.
2. Implementa el flujo mínimo sin optimizar antes de tiempo.
3. Separa UI, estado y acceso a datos si el ejercicio lo justifica.
4. Agrega al menos un caso de error: API falla, formulario inválido, sesión expirada o lista vacía.
5. Escribe una nota corta con lo que aprendiste y lo que cambiarías.

## Criterio de revisión

El laboratorio está completo cuando otra persona puede abrir el código, entender el flujo y ejecutarlo. Revisa nombres, estados de UI, manejo de errores y que no hayas escondido reglas importantes dentro del template.
