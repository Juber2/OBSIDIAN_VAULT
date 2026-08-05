---
tags:
  - dotnet
  - senior
  - review
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Referencia Senior dotNET
  - NET
---

# Guia de Code Review dotNET

## Prioridad de revision

1. Bugs y seguridad.
2. Contratos y datos.
3. Arquitectura y mantenibilidad.
4. Performance observable.
5. Estilo.

## Comentarios utiles

Un comentario debe explicar riesgo, evidencia y alternativa. Si es preferencia personal, dilo como sugerencia, no como bloqueo.

## Preguntas poderosas

¿Que pasa si falla la DB? ¿Hay retry? ¿Es idempotente? ¿Quien valida tenant? ¿Que test falla si rompemos esto?

## Relacionado

- [[Referencia Senior dotNET]]
- [[NET]]
