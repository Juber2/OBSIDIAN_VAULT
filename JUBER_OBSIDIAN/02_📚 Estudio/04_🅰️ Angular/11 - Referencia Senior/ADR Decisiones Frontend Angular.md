---
tags:
  - angular
  - senior
  - adr
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Referencia Senior Angular
  - ANGULAR
---

# ADR Decisiones Frontend Angular

## Cuándo escribir ADR

SSR/CSR, NgRx vs Signals, monorepo Nx, microfrontends, estrategia de auth, design system, testing strategy y migración legacy.

## Formato

Contexto, decisión, alternativas descartadas, consecuencias y fecha. Corto, concreto y mantenible.

## Ejemplo

ADR-004 - Usar Signals para estado local y RxJS para flujos HTTP.

## Valor

Evita discusiones circulares y deja memoria técnica para el equipo.

## Relacionado

- [[Referencia Senior Angular]]
- [[ANGULAR]]

## Uso en proyectos reales

Esta nota funciona como material de consulta durante diseño, implementación y code review. No está pensada para leerse una sola vez, sino para volver cuando tengas que tomar una decisión o revisar una PR. En **ADR Decisiones Frontend Angular**, la clave es convertir criterio técnico en acciones observables.

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
