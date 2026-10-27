---
tags: [ChatGPT, Plugins, Skills, MCP, Connectors]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Plugins, skills, apps, conectores y MCP

## Qué es cada cosa

| Componente | Aporta | Ejemplo |
|---|---|---|
| App/conector | Acceso a un servicio/datos | GitHub, Slack, Figma |
| MCP server | Herramientas mediante protocolo estándar | DB interna, browser, API propia |
| Skill | Conocimiento/procedimiento reusable | Review .NET, release checklist |
| Plugin | Paquete instalable | Skills + MCP + UI/configuración |

La herramienta da capacidad; la skill enseña el procedimiento. Combinarlas suele ser más potente que cualquiera sola.

## Para programación

- GitHub/GitLab: issues, diffs, PRs y repos.
- Figma: diseño estructurado para implementar UI.
- Slack/Linear: convertir conversación/issue en tarea técnica.
- Browser: probar flujo real.
- Docs: consultar fuente oficial actual.
- Security: revisar cambios autorizados.
- Skills de stack: convenciones, templates y comandos.

## Elegir correctamente

Usa conector/app cuando el servicio ya tiene integración mantenida. Usa MCP cuando necesitas una herramienta propia o acceso estándar. Usa skill cuando el problema es «Claude/ChatGPT sabe usar la herramienta, pero no sigue nuestro proceso». Usa plugin cuando quieres distribuir un conjunto completo.

## Permisos

Antes de instalar/conectar:

- datos que puede leer;
- acciones que puede ejecutar;
- workspace/organización;
- almacenamiento de tokens;
- aprobación previa;
- auditoría y revocación;
- reputación/mantenimiento del proveedor.

## Skill de programación útil

```text
Nombre: review-api-net
Trigger: review de endpoints ASP.NET Core
Incluye: auth por objeto, ProblemDetails, DTO/entity mapping, CancellationToken,
EF tracking/N+1, tests, severidades y formato de hallazgo.
```

No metas estas reglas globalmente si solo aplican a .NET; una skill carga el conocimiento cuando corresponde.

## Diseño de combinación

```text
GitHub app → obtiene issue y diff
Figma connector → aporta diseño
Skill frontend → aplica convenciones
Codex → implementa y ejecuta tests
Browser → verifica visualmente
Slack connector → prepara actualización, con aprobación antes de enviar
```

## Riesgos

- prompt injection desde issues/docs;
- permisos de escritura excesivos;
- tools duplicadas/ambiguas;
- costo de contexto;
- plugin desactualizado;
- acciones externas sin revisión.

Fuentes: [Skills & Plugins](https://learn.chatgpt.com/docs/skills-and-plugins), [MCP](https://learn.chatgpt.com/docs/extend/mcp) y [plugins](https://learn.chatgpt.com/docs/plugins).
