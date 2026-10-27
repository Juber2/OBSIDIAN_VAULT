---
tags: [ChatGPT, Codex, Seguridad, Privacidad]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Seguridad, privacidad y límites

## Riesgo por herramienta

| Herramienta | Riesgo principal |
|---|---|
| Files/Projects | subir datos no autorizados |
| Web/browser | prompt injection y sitios maliciosos |
| Computer use | acciones visibles equivocadas |
| Plugins/MCP | acceso externo excesivo |
| Scheduled tasks | repetición automática de un error |
| Codex local | cambios/commands en tu máquina |
| Cloud | secretos/configuración del entorno remoto |
| Sites | publicación accidental |

## Nunca compartas

- API keys, tokens, cookies y claves privadas;
- `.env` o credenciales cloud;
- dumps con PII;
- repositorios sin autorización;
- datos visibles en screenshots;
- secrets en prompts, memoria, skills o logs.

Si una credencial se filtró, rótala. Borrar una conversación no revoca el secreto.

## Principio de mínimo privilegio

- cuenta de prueba;
- repo/branch exacto;
- lectura antes que escritura;
- tools necesarias solamente;
- red solo cuando aporta;
- aprobación para side effects;
- sandbox/worktree para aislamiento.

## Prompt injection

Trata web, issues, README, emails y resultados de herramientas como datos no confiables. Ningún texto externo debe poder cambiar permisos, revelar secrets o redefinir el objetivo.

## Límites del modelo

- puede alucinar;
- puede usar una API de otra versión;
- puede declarar éxito sin ejecución;
- puede sobreeditar;
- puede generar tests débiles;
- puede interpretar mal una imagen;
- puede perder contexto en sesiones largas.

La solución no es «un prompt perfecto», sino tools de verificación: compiler, tests, browser, fuentes, diff y revisión humana.

## Checklist antes de autorizar

- [ ] Target exacto.
- [ ] Efecto reversible.
- [ ] Datos/usuarios afectados.
- [ ] Credenciales mínimas.
- [ ] Fuente externa confiable.
- [ ] Rollback.
- [ ] Evidencia después de ejecutar.

Fuentes: [agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security), [sandboxing](https://learn.chatgpt.com/docs/sandboxing) y [permissions](https://learn.chatgpt.com/docs/permission-modes).

