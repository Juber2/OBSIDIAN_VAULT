---
tags:
  - Angular
  - Senior
  - Testing
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Testing Ecosistema Senior Angular
  - ANGULAR
---

# Proyecto Angular Senior Final

## Objetivo

Construir una app pequeña pero completa: auth, dashboard, CRUD, formularios, API, estado, testing, performance y arquitectura.

## Requisitos

- Standalone components.
- Rutas lazy por feature.
- Auth interceptor + guard.
- Typed reactive forms.
- Signals para estado local.
- RxJS para HTTP/search.
- Tests de componentes/servicios.
- @defer en sección pesada.

## Estructura


```text
src/app/
  core/
  shared/
  features/
    auth/
    users/
    dashboard/
```


## Documentación final

Escribe ADRs cortas: por qué Signals o RxJS, por qué no NgRx, cómo manejas auth, cómo organizas features y qué test cubre cada riesgo.

## Errores comunes

- Hacer solo UI sin datos
- No probar nada
- No documentar decisiones

## Práctica

- [ ] Construir proyecto final y documentar 3 decisiones técnicas

## Relacionado

- [[Testing Ecosistema Senior Angular]]
- [[Angular - Roadmap]]

## Profundización

En **Proyecto Angular Senior Final**, piensa en riesgo. No todas las líneas merecen test, pero todo comportamiento crítico debería estar protegido: login, permisos, formularios, errores HTTP, mappers y navegación principal.

## Estrategia recomendada

- Unit tests para lógica pura, pipes, mappers y validators.
- Component tests para inputs, outputs y DOM visible.
- Integration tests para servicios HTTP, guards e interceptors.
- E2E para flujos críticos: login, CRUD principal y navegación.

## Criterio senior

Un test bueno falla por una regresión real, no por refactor interno. Evita probar detalles privados. Prefiere verificar comportamiento observable: texto visible, evento emitido, request hecha, ruta bloqueada o mensaje de error mostrado.
