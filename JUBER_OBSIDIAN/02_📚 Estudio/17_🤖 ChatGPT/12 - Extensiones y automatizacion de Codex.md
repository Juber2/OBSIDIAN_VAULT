---
tags: [Codex, Skills, Plugins, Hooks, MCP, Subagents]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Extensiones y automatización de Codex

## Skills

Procedimientos/conocimiento reutilizable. Úsalas para tareas repetidas: review de API, migración, documentación, releases o generación con template.

Una skill buena tiene trigger claro, inputs, pasos, validación y límites. Evita una «skill universal» enorme.

## Plugins

Paquetes que pueden combinar skills, MCP servers y UI/configuración. Convienen para distribuir una capacidad entre proyectos/equipo.

## MCP

Conecta herramientas/datos externos. Ejemplos: Figma, GitHub, logs, base de datos read-only, browser.

La combinación típica:

```text
MCP = capacidad de acceder
Skill = conocimiento de cómo usarla
Hook = garantía/automatización alrededor del uso
```

## Hooks

Se activan en eventos y ejecutan una acción determinista. Casos:

- formatear tras edición;
- ejecutar lint/test focalizado;
- bloquear `.env`;
- secret scanning antes de cierre;
- notificar cuando termina una tarea.

Una instrucción puede olvidarse; un hook puede hacer cumplir una regla.

## Subagentes y paralelo

Usa subagentes cuando el trabajo se divide limpiamente:

- mapear backend/frontend por separado;
- revisar seguridad/performance/tests;
- buscar referencias en módulos distintos;
- investigar hipótesis independientes.

No los uses si todos deben editar el mismo archivo o decidir secuencialmente. Más agentes implican más consumo y coordinación.

## SDK, App Server y GitHub Action

- SDK: incorporar Codex programáticamente.
- App Server: construir interfaces/experiencias sobre el agente.
- MCP Server: exponer Codex como capacidad conectable según soporte.
- GitHub Action: ejecutar flujo en CI/PR.
- Non-interactive: scripts y pipelines.

## Madurez por etapas

1. Buen `AGENTS.md`.
2. Una skill repetible.
3. Hook de validación.
4. MCP estrictamente necesario.
5. Plugin para distribuir.
6. Subagentes/paralelo tras medir.
7. CI/SDK cuando el flujo sea estable.

## Evita

- veinte tools siempre conectadas;
- hooks con efectos destructivos;
- skills que instalan paquetes silenciosamente;
- subagentes sin salida acotada;
- plugins sin revisar permisos/código;
- automatizar antes de tener proceso manual confiable.

Fuentes: [build skills](https://learn.chatgpt.com/docs/build-skills), [build plugins](https://learn.chatgpt.com/docs/build-plugins), [hooks](https://learn.chatgpt.com/docs/hooks) y [MCP](https://learn.chatgpt.com/docs/extend/mcp).
