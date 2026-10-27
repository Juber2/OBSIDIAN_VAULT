---
tags: [ClaudeCode, CLI, Desktop, IDE, Web, CI]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Claude Code: superficies e integraciones

Claude Code puede aparecer en terminal, Desktop, IDE, web, Remote Control, Slack y pipelines CI/CD. No todas las funciones son idénticas.

## CLI

Conjunto más completo para terminal, SSH, servidores, scripting y Agent SDK. Ideal si quieres control explícito de directorio, comandos y configuración.

## Desktop Code

Interfaz visual con sesiones locales/SSH, diffs, preview, skills y plugins. Comparte parte de settings con CLI, pero los MCP del chat Desktop y los de Claude Code se configuran separadamente.

## IDE

Contexto de selección/archivos abiertos, edits in-place y revisión junto al código. Ideal para iteración focalizada.

## Web

Tareas remotas sobre repositorios GitHub en entornos que continúan sin la página abierta. Puede preparar dependencias/DB y crear PRs según configuración.

Útil para repos que no tienes localmente o delegación asincrónica. Para cambios locales no committeados, CLI/Desktop/IDE es mejor.

## Remote Control

Supervisar/dirigir una sesión desde otra interfaz/dispositivo manteniendo ejecución en el entorno conectado.

## Slack

Convertir conversación/tarea en ejecución de Claude Code según integración y permisos. Verifica siempre scope, repo y lo que se publicará de vuelta.

## CI/CD y GitHub Actions

Automatizar review, mantenimiento o tareas repetibles. Ejecuta con permisos mínimos, secrets protegidos y output auditable.

## Selección

| Situación | Superficie |
|---|---|
| Terminal/SSH/full control | CLI |
| UI visual + local/SSH | Desktop |
| Código abierto/selección | IDE |
| Tarea remota paralela | Web |
| Supervisión remota | Remote Control |
| Evento de repo | CI/GitHub Action |
| Solicitud desde equipo | Slack |

## Combinar

IDE para definir, web para delegar, Desktop para revisar y CLI para validación final. No dupliques el mismo cambio en varias superficies sin branches/worktrees aislados.

Fuentes: [plataformas Claude Code](https://code.claude.com/docs/en/platforms), [Desktop](https://code.claude.com/docs/en/desktop) y [Claude Code web](https://support.claude.com/en/articles/12618689-claude-code-on-the-web).

