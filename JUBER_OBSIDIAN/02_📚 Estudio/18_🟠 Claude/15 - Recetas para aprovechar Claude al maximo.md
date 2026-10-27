---
tags: [Claude, ClaudeCode, Recetas, Prompts]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Recetas para aprovechar Claude al máximo

## 1. Feature desde requisitos

```text
Project → requisitos/arquitectura
Connector → issue/diseño
Artifact → validar UX
Claude Code IDE/Desktop → implementar
Computer use → QA visual
GitHub connector → PR, con aprobación
```

## 2. Bug complejo

```text
Files/code execution → analizar logs
Extended thinking → hipótesis
Web search → docs/CVEs actuales
Claude Code → reproducir/test/fix
Cowork/computer use → retest UI
```

## 3. Migración

```text
Research → breaking changes oficiales
Project/RAG → arquitectura y corpus
Claude Code web/subagents → lotes aislados
Hooks → tests/lint obligatorios
CLI → validación final
```

## 4. Revisión de PR

```text
GitHub connector → diff/contexto
Skill → checklist del stack
Subagents → seguridad/performance/tests
Artifact → walkthrough visual
Humano → valida antes de comentar/merge
```

## 5. Automatización de equipo

```text
Cowork task → trigger/horario
Connector → fuente
Skill → procedimiento
Artifact → dashboard
Approval → escritura externa
```

## 6. Project ideal

```text
Project knowledge: ADRs + OpenAPI + schema + glossary
Project instructions: stack + autoridad + output
RAG: corpus grande y versionado
Memory: preferencias globales, no estado técnico
Claude Code: repo y validación real
```

## Prompt mínimo universal

```text
Objetivo: [resultado].
Contexto/tools: [Project/files/web/connector/repo].
Puede: [leer/editar/ejecutar].
Debe pedir permiso: [red/publicar/dependencias/borrar].
Éxito: [criterios + checks].
Salida: [artifact/diff/reporte/archivo].
```

## Prompt estructurado para Claude

```text
<objective>[resultado]</objective>
<sources>[Project, archivos y connectors autorizados]</sources>
<constraints>[alcance, permisos y versiones]</constraints>
<verification>[tests, build, fuentes o revisión]</verification>
<deliverable>[formato final]</deliverable>
```

XML ayuda cuando hay varios bloques; no es necesario en una petición corta.

## Claude vs ChatGPT/Codex

No elijas por marca. Compara la herramienta equivalente:

- Project vs Project;
- Artifact vs Sites/Visualizations;
- Cowork/computer use vs Work/computer use;
- Claude Code CLI/IDE/web vs Codex CLI/IDE/cloud;
- Skills/MCP/hooks/plugins en cada ecosistema.

Mide éxito, correcciones, tiempo, consumo y seguridad con el mismo repo/tarea. Consulta [[CHATGPT]] para el mapa del otro ecosistema.

## Checklist

- [ ] Superficie adecuada.
- [ ] Tool mínima necesaria.
- [ ] Fuente de verdad identificada.
- [ ] Permisos mínimos.
- [ ] Salida revisable.
- [ ] Evidencia real.
- [ ] Consumo proporcional.
- [ ] Conocimiento durable guardado en Project/repo/Skill.

Volver: [[CLAUDE PARA PROGRAMAR]].

