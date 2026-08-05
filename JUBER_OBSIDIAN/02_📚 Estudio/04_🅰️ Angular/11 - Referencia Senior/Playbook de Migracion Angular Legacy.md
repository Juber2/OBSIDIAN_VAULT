---
tags:
  - angular
  - senior
  - legacy
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Referencia Senior Angular
  - ANGULAR
---

# Playbook de Migracion Angular Legacy

## Orden seguro

1. Congelar comportamiento con tests mínimos.
2. Actualizar Angular por pasos.
3. Migrar NgModules a standalone gradualmente.
4. Cambiar templates a control flow moderno donde convenga.
5. Introducir Signals en estado local nuevo.

## Riesgo

No migres todo en una rama gigante. Parte por features y conserva compatibilidad.

## Criterio

Legacy no se arregla solo con sintaxis nueva. También hay que reducir componentes grandes, servicios dios y dependencias cruzadas.

## Relacionado

- [[Referencia Senior Angular]]
- [[ANGULAR]]

## Uso en proyectos reales

Esta nota funciona como material de consulta durante diseño, implementación y code review. No está pensada para leerse una sola vez, sino para volver cuando tengas que tomar una decisión o revisar una PR. En **Playbook de Migracion Angular Legacy**, la clave es convertir criterio técnico en acciones observables.

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
