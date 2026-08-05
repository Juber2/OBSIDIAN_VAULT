---
tags:
  - angular
  - senior
  - checklist
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Referencia Senior Angular
  - ANGULAR
---

# Checklist Profesional Angular

## Arquitectura

Features con boundaries, shared sin dominio, core para singletons, rutas lazy, imports claros y aliases con reglas.

## UI y templates

Estados loading/error/empty/success, `track` en listas, accesibilidad básica, componentes pequeños y sin lógica pesada en template.

## Datos y estado

DTO mapping, servicios data-access, RxJS para eventos, Signals para estado actual, NgRx solo si aporta valor real.

## Seguridad

Guards para UX, backend como autoridad, tokens con estrategia, sanitización, no exponer secretos, no confiar permisos al cliente.

## Calidad

Tests donde hay riesgo, E2E para flujos críticos, budgets, lint, build reproducible y code review.

## Relacionado

- [[Referencia Senior Angular]]
- [[ANGULAR]]

## Uso en proyectos reales

Esta nota funciona como material de consulta durante diseño, implementación y code review. No está pensada para leerse una sola vez, sino para volver cuando tengas que tomar una decisión o revisar una PR. En **Checklist Profesional Angular**, la clave es convertir criterio técnico en acciones observables.

## Señales de buena aplicación

- La decisión queda escrita y se entiende el contexto.
- El código resultante reduce riesgo, no solo cambia estilo.
- Hay un responsable claro para mantener la decisión.
- El equipo puede explicar el tradeoff sin depender de una persona.

## Señales de mala aplicación

- Se usa como checklist mecánico sin pensar.
- Se bloquean PRs por preferencias personales.
- Se agregan patrones sin problema real.
- Se documenta mucho, pero el código sigue sin tests ni observabilidad.

## Pregunta de cierre

Antes de aplicar esta referencia, pregúntate: ¿qué riesgo concreto estoy reduciendo? Si no puedes responderlo, quizá estás agregando proceso y no calidad.
