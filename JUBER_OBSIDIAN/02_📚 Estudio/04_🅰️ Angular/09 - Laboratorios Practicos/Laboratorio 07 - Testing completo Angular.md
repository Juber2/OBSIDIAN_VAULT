---
tags:
  - angular
  - laboratorio
  - testing
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos Angular
  - ANGULAR
---

# Laboratorio 07 - Testing completo Angular

## Objetivo

Crear pruebas útiles para componentes, servicios HTTP, guards, interceptors y flujos críticos.

## Suite mínima

- Test de componente presentacional.
- Test de servicio HTTP con mock.
- Test de formulario y validaciones.
- Test de guard.
- E2E del login y navegación principal.

## Criterio

El test debe romperse si falla comportamiento importante, no si renombraste una variable interna.

## Ejemplo component test


```ts
fixture.componentRef.setInput("user", user);
fixture.detectChanges();
expect(screen.getByText(user.name)).toBeTruthy();
```


## Errores comunes

- Tests que repiten implementación
- No probar errores
- E2E para todo

## Practica

- [ ] Crear test de UserCard
- [ ] Crear test de AuthGuard
- [ ] Crear E2E de login

## Relacionado

- [[Laboratorios Practicos Angular]]
- [[ANGULAR]]

## Desarrollo guiado

Este laboratorio no se debe resolver copiando snippets aislados. La idea es construir una pequeña pieza funcional, ejecutar la app y registrar qué decisiones tomaste. Para **Laboratorio 07 - Testing completo Angular**, trabaja en tres capas: primero haz que funcione, luego ordena responsabilidades y finalmente agrega validación o prueba mínima.

## Pasos sugeridos

1. Crea una rama o carpeta de práctica con nombre claro.
2. Implementa el flujo mínimo sin optimizar antes de tiempo.
3. Separa UI, estado y acceso a datos si el ejercicio lo justifica.
4. Agrega al menos un caso de error: API falla, formulario inválido, sesión expirada o lista vacía.
5. Escribe una nota corta con lo que aprendiste y lo que cambiarías.

## Criterio de revisión

El laboratorio está completo cuando otra persona puede abrir el código, entender el flujo y ejecutarlo. Revisa nombres, estados de UI, manejo de errores y que no hayas escondido reglas importantes dentro del template.
