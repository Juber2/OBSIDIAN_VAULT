---
tags:
  - Angular
  - Senior
  - Senior
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Arquitectura Performance Seguridad Angular
  - ANGULAR
---

# Feature Architecture Boundaries

## Feature based

Organiza por capacidades de negocio: users, auth, dashboard, courses.

## Estructura


```text
features/users/
  users.routes.ts
  pages/
  ui/
  data-access/
  models/
```


## Core Shared Features

Core: singletons/global. Shared: UI/utilidades sin dominio. Features: negocio.

## Boundaries

Una feature no debe importar detalles internos de otra. Si necesita algo, expón API pública.

## Aliases

Ayudan a imports, pero no reemplazan reglas. En Nx puedes hacer enforce module boundaries.

## Errores comunes

- Carpetas lindas sin reglas
- Shared con dominio
- Imports cruzados entre features

## Práctica

- [ ] Diseñar estructura para sistema administrativo

## Relacionado

- [[Arquitectura Performance Seguridad Angular]]
- [[Angular - Roadmap]]

## Decisión técnica

En **Feature Architecture Boundaries**, no busques una receta única. Angular senior consiste en elegir estructura según tamaño del equipo, vida útil del producto, cantidad de features, necesidad de SEO, performance, permisos y testing.

## Preguntas antes de decidir

- ¿Este código cambiará por negocio o por infraestructura?
- ¿La feature tiene ownership claro?
- ¿Necesita estado compartido o solo local?
- ¿Hay rutas lazy y boundaries?
- ¿Qué parte debe poder testearse sin levantar toda la app?

## Criterio de madurez

Una arquitectura buena no se ve solo en carpetas bonitas. Se ve cuando agregar una feature no rompe otra, cuando una PR es revisable, cuando los tests cubren riesgos y cuando el equipo entiende dónde poner cada pieza.
