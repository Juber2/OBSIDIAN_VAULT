---
tags: [Claude, ClaudeCode, Cowork, Artifacts, Herramientas]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Mapa de Claude y sus superficies

## Claude chat

Conversación para explicar, analizar, investigar y crear. Puede usar archivos, web, code execution, skills, connectors y Artifacts según configuración.

**Programación:** especificaciones, revisión conceptual, explicación de código, análisis de archivos y documentación.

## Projects

Agrupan chats, instrucciones y base de conocimiento. Cuando el contenido crece, RAG puede recuperar partes relevantes sin cargar todo directamente.

**Programación:** un Project por producto con ADRs, OpenAPI, README, decisiones y documentación.

## Artifacts

Ventana separada con contenido autocontenido: código, HTML/React, SVG, Mermaid, documentos y aplicaciones interactivas.

**Programación:** prototipo UI, diagrama, herramienta interna, visualizador o demo.

## Cowork

Superficie de trabajo agéntico sobre archivos, conectores, apps y, cuando está disponible, computer use. Sus Projects pueden incluir contexto, instrucciones, memoria y tareas programadas.

**Programación:** organizar materiales, preparar reportes, operar herramientas y flujos que cruzan aplicaciones.

## Claude Code

Agente de ingeniería con herramientas de filesystem, búsqueda, edición, shell, web y extensiones. Disponible en varias superficies.

**Programación:** cambios multiarchivo, tests, debugging, review, migraciones y automatización.

## Matriz

| Superficie | Contexto principal | Puede actuar | Resultado típico |
|---|---|---|---|
| Chat | Conversación/adjuntos | Herramientas habilitadas | Respuesta/archivo/artifact |
| Project | Conocimiento + chats | Igual que chat | Trabajo continuo |
| Artifact | Artefacto autocontenido | Código/IA/MCP según configuración | Prototipo/app/documento |
| Cowork | Archivos/apps/conectores | Sí, con permisos | Tarea operativa |
| Claude Code | Repositorio/terminal | Sí | Diff + validación |

## Elección práctica

- Explicar → Chat.
- Mantener contexto → Project.
- Mostrar/iterar algo visual → Artifact.
- Mover trabajo entre apps/archivos → Cowork.
- Editar repo/ejecutar tests → Claude Code.
- Consultar servicio → Connector/MCP.
- Repetir proceso → Skill.

## Error común

Pedirle al chat que «edite mi proyecto» cuando solo recibió dos archivos pegados. O usar Claude Code para una simple comparación conceptual. La superficie correcta suele importar tanto como el modelo.

Fuente: [capacidades de Claude](https://support.claude.com/en/collections/18031719-features-and-capabilities) y [plataformas Claude Code](https://code.claude.com/docs/en/platforms).

