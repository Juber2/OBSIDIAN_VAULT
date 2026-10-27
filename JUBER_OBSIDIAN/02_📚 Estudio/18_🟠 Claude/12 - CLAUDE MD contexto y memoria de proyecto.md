---
tags: [ClaudeCode, CLAUDEMD, Contexto, Compaction]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# `CLAUDE.md`, contexto y memoria de proyecto

## `CLAUDE.md`

Archivo de instrucciones persistentes que Claude Code carga por scope. Puede existir a nivel usuario, repo y subdirectorios; las capas son aditivas.

## Qué incluir

```markdown
# Project guide
## Purpose
## Architecture and boundaries
## Commands: setup/build/test/lint
## Conventions not enforced by tooling
## Generated/sensitive paths
## Approval boundaries
## Definition of done
```

## Qué no incluir

- secretos;
- README completo;
- documentación de 500 líneas;
- estado de un issue;
- ejemplos repetitivos;
- reglas que deberían vivir en formatter/hook;
- referencia especializada que debería ser Skill.

La guía actual de extensiones recomienda mantener `CLAUDE.md` bajo aproximadamente 200 líneas y mover material on-demand a Skills.

## Context window

Consume:

- conversación;
- `CLAUDE.md`;
- outputs de tools;
- skills cargadas;
- esquemas MCP;
- archivos leídos.

Usa `/context` para inspeccionar. Desconecta MCP no usados y evita outputs gigantes.

## Compaction

Resume contexto cuando se llena. Si una sesión empieza a confundir decisiones, crea un handoff y abre una limpia. Archivos enormes pueden causar thrashing; reduce/filtra la fuente.

## Skills y subagentes para contexto

- Skill: descripción ligera, contenido al invocar.
- Subagent: ventana aislada y devuelve resumen.
- MCP: tool names y schemas diferidos según configuración.
- Hook: no consume contexto salvo output.

## Imports/reglas anidadas

Usa archivos más específicos en subdirectorios/monorepo solo cuando las reglas realmente cambian. Conflictos entre capas fuerzan al modelo a reconciliar instrucciones y degradan consistencia.

## Plantilla mínima

```markdown
# Commands
- Backend tests: `dotnet test`
- Frontend tests: `npm test -- --watch=false`
- Build: `npm run build && dotnet build`

# Boundaries
- Do not edit generated clients.
- Ask before dependencies, migrations, network writes or deploys.

# Done
- Targeted tests, build, diff review, factual summary.
```

Fuentes: [contexto con `CLAUDE.md`](https://support.claude.com/en/articles/14553240-give-claude-context-claude-md-and-better-prompts) y [extensiones/context costs](https://code.claude.com/docs/en/features-overview).

