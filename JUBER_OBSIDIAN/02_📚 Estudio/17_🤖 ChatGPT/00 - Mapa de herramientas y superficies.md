---
tags: [ChatGPT, Codex, Herramientas]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Mapa de herramientas y superficies

El ecosistema actual separa tres formas principales de trabajo:

## Chat

Conversación interactiva para explicar, pensar, comparar, investigar y producir borradores. Puede combinar archivos, búsqueda web, imágenes, plugins y otras capacidades habilitadas.

**Aprovéchalo para:**

- entender un concepto o fragmento;
- convertir una idea en especificación;
- comparar tecnologías;
- revisar un error o diseño con los archivos relevantes;
- preparar el encargo que después ejecutará Codex.

**No es ideal para:** cambiar un repositorio grande mediante copiar/pegar manual o mantener un flujo largo con muchos comandos reales.

## Work

Modo para llevar una tarea más grande hasta un resultado revisable. Es apropiado cuando el entregable importa más que el diálogo: informe, análisis, documento, visualización, paquete de archivos o flujo que usa varias herramientas.

**Aprovéchalo para programación:**

- analizar exports de logs o métricas;
- crear documentación desde varias fuentes;
- producir una visualización interactiva de arquitectura;
- preparar una especificación, matriz de pruebas o informe de incidentes;
- realizar trabajo largo apoyado en archivos y plugins.

## Codex

Superficie de ingeniería: inspecciona repositorios, edita archivos, ejecuta comandos y mantiene tareas técnicas. Puede presentarse en Desktop, CLI, IDE, cloud y Remote.

**Aprovéchalo para:**

- features multiarchivo;
- debugging con tests/logs;
- refactors y migraciones;
- code review;
- ejecución de build, lint y pruebas;
- automatización de mantenimiento.

## Matriz por superficie

| Superficie | Contexto | Ejecuta/edita | Uso ideal |
|---|---|---|---|
| Chat web/móvil | Conversación, archivos, herramientas | Según capacidad activada | Consulta y colaboración |
| Desktop | Proyectos, archivos locales y tareas paralelas | Sí, con permisos | Centro de trabajo diario |
| Codex CLI | Directorio/terminal | Sí | Desarrolladores orientados a shell |
| Codex IDE | Selección, archivos abiertos y repo | Sí | Iteración junto al editor |
| Codex cloud | Repo y entorno reproducible | Sí, aislado/remoto | Delegación y paralelismo |
| Remote | Equipo conectado | Guía tareas del equipo | Supervisión móvil |

## Regla práctica

Empieza por la superficie más pequeña que pueda completar el resultado:

- pregunta → Chat;
- artefacto complejo → Work;
- código real → Codex;
- actividad periódica → Scheduled task;
- servicio externo → plugin/app/MCP;
- UI real → browser/computer use;
- sitio alojado → Sites.

## Cómo no desperdiciar herramientas

No actives todo siempre. Más herramientas implican más permisos, ruido y posibilidades de usar la capacidad equivocada. Para cada tarea define:

1. Fuente de verdad.
2. Herramienta que puede leerla.
3. Resultado concreto.
4. Acciones autorizadas.
5. Evidencia requerida.

Fuente: [guía oficial para usar ChatGPT](https://learn.chatgpt.com/docs/use-chatgpt).

