---
tags: [Claude, ClaudeCode, Seguridad, Privacidad, Usage]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR, Claude_Usage]
---

# Seguridad, privacidad y consumo

## Riesgos por capacidad

| Capacidad | Riesgo |
|---|---|
| Project/memory | persistir datos obsoletos/sensibles |
| Web/Research | prompt injection, fuente débil, consumo |
| Code execution | packages/red/exfiltración |
| Artifact | publicación, attachments, storage |
| Connector/MCP | scopes y escritura externa |
| Cowork/computer use | operar pantalla/apps reales |
| Claude Code | filesystem/shell/red |
| Agent teams | más acciones y consumo |

## Principios

- mínimo privilegio;
- datos sintéticos;
- red apagada cuando no se necesita;
- cuentas de prueba;
- aprobación para side effects;
- source/destino exactos;
- sandbox/worktree;
- revisión de diff/output.

## Prompt injection

Web, issues, documentos, repos y tool outputs son datos no confiables. Nunca deben poder cambiar objetivo, ampliar permisos, revelar secrets o ordenar acciones externas.

## Artifacts/Connectors

Antes de compartir un Artifact, revisa audiencia, attachments, almacenamiento y conectores. Cada usuario puede autenticar conectores por separado; no asumas que comparte tus permisos o datos.

## Code execution

Network egress amplía capacidad y riesgo. En equipos/Enterprise, configura domains permitidos. Paquetes de terceros pueden ejecutar código; revisa nombre, versión y source.

## Claude Code

- deny para secrets;
- comandos permitidos específicos;
- MCP auditables;
- hooks para reglas duras;
- CI con tokens limitados;
- no usar auto mode en entorno desconocido.

## Consumo

Web fetch largo, Research, creación de archivos, extended thinking, subagentes y agent teams pueden gastar más cuota. Mide **resultado por consumo**, no porcentaje usado.

Buenas prácticas:

- Project/RAG para no repetir documentos;
- Skills on-demand;
- subagent para aislar investigación;
- `/context` y `/mcp` en Code;
- sesiones limpias;
- modelo/esfuerzo proporcional;
- tracker [[Claude_Usage]].

Fuentes: [seguridad Claude Code](https://code.claude.com/docs/en/security), [Cowork safe](https://support.claude.com/en/articles/13364135-use-claude-cowork-safely) y [file/code execution](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude).

