---
tags: [Codex, AGENTS, Permissions, Sandbox, Rules]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# `AGENTS.md`, memoria, reglas, permisos y sandbox

## `AGENTS.md`

Guarda instrucciones del repositorio que Codex debe conocer: arquitectura, comandos, convenciones, límites y definición de terminado.

```markdown
# Repository guide

## Architecture
- Dependencias permitidas y directorios clave.

## Commands
- Setup, build, tests focalizados, lint.

## Conventions
- Reglas no automatizadas y ejemplos.

## Boundaries
- Generados/no tocar.
- Dependencias, migraciones y acciones externas requieren aprobación.

## Done
- Tests + build + diff review + docs cuando aplique.
```

No guardes secrets, estado temporal ni manuales enteros. Usa rules/skills para contenido especializado.

## Memorias y reglas

- Memoria: aprendizajes/preferencias persistentes del agente.
- `AGENTS.md`: contrato del repositorio.
- Rules: instrucciones por scope/patrón.
- Prompt: requisito de la tarea actual.
- Skill: proceso reusable cargado cuando aplica.

Coloca cada regla en el nivel más estrecho que corresponda.

## Modos

Usa modo de solo lectura/plan para comprender. Usa modo de edición normal para trabajo local esperado. Mayor autonomía solo con entorno aislado y criterios claros.

## Permisos

Define qué puede hacer en tu equipo y cuándo pide aprobación. Buenas fronteras:

- lectura y búsqueda: permitidas;
- edits en scope: permitidos;
- tests/build: permitidos;
- instalar dependencias: aprobación;
- red/escritura externa: aprobación;
- deploy/merge/publicar: aprobación;
- borrar/mover datos: aprobación.

## Sandbox

Reduce daño aislando filesystem, red y comandos. No es excusa para confiar ciegamente: revisa targets y outputs. En Windows/WSL entiende dónde corre realmente la herramienta y qué rutas puede alcanzar.

## Worktrees

Úsalos para tareas paralelas sin mezclar cambios. Cada agente/tarea trabaja en rama/copia aislada. Verifica destino al integrar y elimina worktrees solo cuando cambios estén preservados.

## Configuración práctica

- Permite prefijos específicos (`npm test`, `dotnet test`).
- Niega archivos secretos/generados.
- Mantén red apagada salvo necesidad.
- Usa auto-review antes de acciones sensibles.
- Revisa reglas compartidas en PR.
- Audita permisos tras instalar plugins.

Fuentes: [`AGENTS.md`](https://learn.chatgpt.com/docs/agent-configuration/agents-md), [permissions](https://learn.chatgpt.com/docs/permission-modes), [sandboxing](https://learn.chatgpt.com/docs/sandboxing) y [worktrees](https://learn.chatgpt.com/docs/environments/git-worktrees).
