---
tags: [Claude, Cowork, ComputerUse, ScheduledTasks]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Cowork, computer use y tareas

## Cowork

Entorno agéntico de Claude Desktop orientado a tareas sobre contexto, archivos y apps. Sus Projects pueden incluir instrucciones, contexto local/URLs, memoria y tareas programadas según disponibilidad.

## Computer use

Cuando no hay connector/tool preciso, Claude puede navegar pantalla, hacer clic, escribir y abrir apps. En programación:

- probar una app visualmente;
- operar herramientas sin API;
- recopilar screenshots;
- ejecutar flujos en dev tools;
- mover información entre aplicaciones autorizadas.

La documentación indica que connector es preferido, luego herramientas más precisas, y computer use se usa cuando hace falta interfaz visual.

## Riesgo especial

Computer use interactúa con lo visible en pantalla y puede no tener el aislamiento de code execution. Usa:

- cuenta de prueba;
- datos aislados;
- confirmación para enviar/publicar/borrar;
- ventanas sin información sensible;
- alcance y criterios exactos;
- supervisión en acciones de alto impacto.

## Cowork Projects

Úsalos para agrupar:

- carpeta local;
- Project de chat;
- URLs/contexto;
- instrucciones;
- memoria;
- tareas recurrentes.

No confundas Projects de Cowork con Claude Code ni con Projects de chat: comparten la idea, pero su almacenamiento/superficie puede variar.

## Tareas programadas

Programación:

- revisar issues/changelogs;
- compilar resumen semanal;
- preparar reporte de incidentes;
- monitorear PRs;
- actualizar dashboard live Artifact.

Automatiza primero lectura/reporte. Escritura externa, deploy o borrado siempre con aprobación.

## Flujo QA

```text
Cowork Project → contexto y cuenta de prueba
Computer use → ejecutar flujo UI
Artifact → timeline/informe visual
Claude Code → diagnosticar/corregir repo
Computer use → retest
```

Fuentes: [computer use en Cowork](https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork), [seguridad Cowork](https://support.claude.com/en/articles/13364135-use-claude-cowork-safely) y [Projects de Cowork](https://support.claude.com/en/articles/14116274-organize-your-tasks-with-projects-in-claude-cowork).

