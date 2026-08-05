---
tags:
  - angular
  - laboratorio
  - base
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos Angular
  - ANGULAR
---

# Laboratorio 01 - App base standalone

## Objetivo

Crear una app Angular moderna desde cero usando standalone components, rutas raíz, estructura por features y convenciones de nombres profesionales.

## Entregables

- App creada con Angular CLI.
- `app.config.ts` con providers globales.
- `app.routes.ts` con rutas base.
- Carpetas `core`, `shared` y `features`.
- Feature `dashboard` lazy.

## Estructura sugerida


```text
src/app/
  core/
  shared/
  features/
    dashboard/
      dashboard.routes.ts
      pages/
      ui/
      data-access/
```


## Criterio de terminado

La app compila, navega a dashboard, y cada archivo tiene nombre kebab-case con clase PascalCase.

## Errores comunes

- Crear todo en app/
- Usar NgModules sin necesidad
- No separar feature de shared

## Practica

- [ ] Crear app
- [ ] Agregar ruta lazy
- [ ] Documentar estructura

## Relacionado

- [[Laboratorios Practicos Angular]]
- [[ANGULAR]]

## Desarrollo guiado

Este laboratorio no se debe resolver copiando snippets aislados. La idea es construir una pequeña pieza funcional, ejecutar la app y registrar qué decisiones tomaste. Para **Laboratorio 01 - App base standalone**, trabaja en tres capas: primero haz que funcione, luego ordena responsabilidades y finalmente agrega validación o prueba mínima.

## Pasos sugeridos

1. Crea una rama o carpeta de práctica con nombre claro.
2. Implementa el flujo mínimo sin optimizar antes de tiempo.
3. Separa UI, estado y acceso a datos si el ejercicio lo justifica.
4. Agrega al menos un caso de error: API falla, formulario inválido, sesión expirada o lista vacía.
5. Escribe una nota corta con lo que aprendiste y lo que cambiarías.

## Criterio de revisión

El laboratorio está completo cuando otra persona puede abrir el código, entender el flujo y ejecutarlo. Revisa nombres, estados de UI, manejo de errores y que no hayas escondido reglas importantes dentro del template.
