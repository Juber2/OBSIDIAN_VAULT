---
tags: [ClaudeCode, Skills, Subagents, AgentTeams, Hooks, MCP, Plugins]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Extensiones de Claude Code

## Mapa

| Extensión | Función | Cuándo |
|---|---|---|
| `CLAUDE.md` | Contexto siempre cargado | Convenciones permanentes |
| Skill | Conocimiento/workflow on-demand | Tarea reusable |
| MCP | Tools y datos externos | DB, Slack, browser, APIs |
| Subagent | Worker aislado | Investigación/contexto separado |
| Agent team | Sesiones coordinadas | Paralelo con comunicación |
| Hook | Evento determinista | Lint, bloqueo, logging |
| Plugin | Empaquetar/distribuir | Reusar setup completo |

## Subagents

Contexto propio, no heredan toda la conversación; devuelven resumen. Úsalos para leer muchos archivos, revisar un área o comprobar una hipótesis.

## Agent teams

Múltiples sesiones coordinadas con tareas/mensajería. Útiles cuando deben compartir hallazgos o desafiarse: security/performance/tests o feature con componentes independientes. Función experimental según documentación; mayor consumo.

## Hooks

Eventos de lifecycle ejecutan command, HTTP, prompt o subagent según soporte. Usa command hook para garantías deterministas:

- bloquear `.env`;
- ejecutar formatter;
- secret scan;
- impedir comando peligroso;
- notificar al finalizar.

Un prompt «nunca edites X» es una petición; un `PreToolUse` hook puede ser enforcement.

## MCP + Skill

MCP conecta base de datos; Skill aporta esquema, consultas aprobadas y política read-only. La combinación reduce improvisación.

## Plugins

Empaquetan skills, agents, hooks, MCP, LSP y otros componentes. Standalone `.claude/` sirve para personal/proyecto; plugin conviene para versionar/distribuir.

## Orden de adopción

1. `CLAUDE.md` breve.
2. Skill de flujo repetido.
3. Hook de validación.
4. MCP necesario.
5. Subagent para aislamiento.
6. Plugin para compartir.
7. Agent team solo tras medir beneficio.

## Context cost

Todo agrega ruido. Descripciones de Skills, tools MCP y reglas compiten por atención. Usa activación manual para side effects, desconecta MCP ociosos y limita agentes.

Fuentes: [feature overview](https://code.claude.com/docs/en/features-overview), [hooks](https://code.claude.com/docs/en/hooks), [plugins](https://code.claude.com/docs/en/plugins) y [tools reference](https://code.claude.com/docs/en/tools-reference).

