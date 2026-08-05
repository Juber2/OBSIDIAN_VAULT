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

# Nx Monorepos Microfrontends

## Nx

Útil para múltiples apps/libs, cache, affected commands y boundaries.

## Tipos de librerías

ui, data-access, feature, util, domain. Cada una debe tener propósito claro.

## Microfrontends

Útiles si equipos/domínios necesitan despliegue independiente real. No son solución para carpetas desordenadas.

## Comandos Nx


```bash
nx graph
nx affected -t test build lint
```


## Module Federation

Permite cargar remotes en runtime, pero agrega riesgos de versiones, shared deps y coordinación.

## Errores comunes

- Nx para proyecto pequeño sin razón
- Microfrontends por moda
- Dependencias compartidas incompatibles

## Práctica

- [ ] Decidir si una app hipotética necesita Nx o no

## Relacionado

- [[Testing Ecosistema Senior Angular]]
- [[Angular - Roadmap]]

## Profundización

En **Nx Monorepos Microfrontends**, piensa en riesgo. No todas las líneas merecen test, pero todo comportamiento crítico debería estar protegido: login, permisos, formularios, errores HTTP, mappers y navegación principal.

## Estrategia recomendada

- Unit tests para lógica pura, pipes, mappers y validators.
- Component tests para inputs, outputs y DOM visible.
- Integration tests para servicios HTTP, guards e interceptors.
- E2E para flujos críticos: login, CRUD principal y navegación.

## Criterio senior

Un test bueno falla por una regresión real, no por refactor interno. Evita probar detalles privados. Prefiere verificar comportamiento observable: texto visible, evento emitido, request hecha, ruta bloqueada o mensaje de error mostrado.
