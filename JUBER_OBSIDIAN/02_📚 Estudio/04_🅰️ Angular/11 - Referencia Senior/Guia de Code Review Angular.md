---
tags:
  - angular
  - senior
  - review
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Referencia Senior Angular
  - ANGULAR
---

# Guia de Code Review Angular

## Prioridad

1. Bugs y seguridad.
2. Estado y datos.
3. Arquitectura y boundaries.
4. Performance y accesibilidad.
5. Estilo.

## Preguntas

¿Quién owns este estado? ¿Qué pasa si falla HTTP? ¿Hay loading/error/empty? ¿Se limpia suscripción? ¿Hay `track`? ¿El backend valida permiso?

## Comentario útil

Debe explicar riesgo y alternativa. Ejemplo: “Este componente hace HTTP y mapping. Movamos esa lógica a data-access para testearla y reducir responsabilidad.”

## Relacionado

- [[Referencia Senior Angular]]
- [[ANGULAR]]

## Uso en proyectos reales

Esta nota funciona como material de consulta durante diseño, implementación y code review. No está pensada para leerse una sola vez, sino para volver cuando tengas que tomar una decisión o revisar una PR. En **Guia de Code Review Angular**, la clave es convertir criterio técnico en acciones observables.

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
