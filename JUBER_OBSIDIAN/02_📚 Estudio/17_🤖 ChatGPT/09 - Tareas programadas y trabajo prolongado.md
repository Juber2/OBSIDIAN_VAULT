---
tags: [ChatGPT, Automations, ScheduledTasks, LongRunning]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Tareas programadas y trabajo prolongado

## Scheduled tasks

Ejecutan tareas recurrentes en segundo plano. Pueden combinarse con skills y herramientas conectadas.

### Programación

- resumen diario de issues/PRs;
- revisión semanal de dependencias;
- actualización de documentación;
- reporte de tests nocturnos;
- triage de bugs;
- vigilancia de changelogs/CVEs;
- recordatorio de deuda o releases.

## Local vs web

| Tipo | Puede usar |
|---|---|
| Desktop/local | Proyecto/directorio local o worktree; equipo/app deben estar disponibles |
| Web | Contexto subido y herramientas conectadas; no carpeta local directamente |

Esto determina si la tarea puede ejecutar el build real o solo analizar fuentes remotas.

## Diseño de una automatización

Define:

- horario/zona;
- fuente de datos;
- scope;
- skill/proceso;
- salida esperada;
- condición de éxito;
- notificación;
- acción que requiere aprobación;
- política ante fallos/reintentos.

## Ejemplo seguro

```text
Cada lunes 08:00 America/Lima, revisa changelogs oficiales de Angular, .NET y NestJS
publicados en los últimos 7 días. Compara con versiones del archivo versions.md.
Genera un reporte con impacto y fuentes. No actualices dependencias ni abras PRs.
Notifícame solo si existe cambio relevante o fallo de ejecución.
```

## Long-running work

Para objetivos que duran varias iteraciones: migración, auditoría extensa, refactor grande o investigación. Mantén:

- objetivo durable;
- hitos;
- archivos de estado;
- validación por checkpoint;
- presupuesto/tiempo;
- regla de parada/bloqueo.

## Notifications

Úsalas para:

- resultado terminado;
- aprobación requerida;
- error;
- cambio relevante;
- límite alcanzado.

Evita notificaciones por cada paso; configura señal útil, no ruido.

## Guardrails

- Automatiza lectura/reporte antes que escritura.
- No permitas deploy/merge/borrado periódico sin aprobación.
- Usa worktree/entorno aislado.
- Registra fuentes y comandos.
- Revisa la primera semana de ejecuciones.
- Desactiva tareas obsoletas.

Fuentes: [Scheduled tasks](https://learn.chatgpt.com/docs/automations) y [long-running work](https://learn.chatgpt.com/docs/long-running-work).

