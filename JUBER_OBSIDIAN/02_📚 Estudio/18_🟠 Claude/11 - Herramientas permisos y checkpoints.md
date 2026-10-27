---
tags: [ClaudeCode, Tools, Permissions, Checkpoints]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Herramientas built-in, permisos y checkpoints

## Categorías de tools

Claude Code incluye herramientas para:

- leer/buscar archivos y patrones;
- editar/escribir;
- ejecutar shell;
- web search/fetch;
- preguntar al usuario;
- lanzar subagentes;
- notebooks/recursos según entorno;
- usar skills y tools MCP.

Los nombres exactos importan en reglas de permisos, subagentes y hooks. Consulta el reference actual.

## Permission modes

Elige por riesgo:

- plan/solo lectura para explorar;
- permisos normales para edits y checks locales;
- auto/autonomía solo con sandbox, alcance y entorno controlado;
- deny explícito para secrets/acciones peligrosas.

## Reglas de allow/deny

Permite comandos concretos (`npm test`, `dotnet build`) en vez de Bash ilimitado. Niega `.env`, keys, producción y comandos destructivos.

## Checkpoints/rewind

Permiten volver a estados anteriores de conversación/cambios según superficie. Úsalos antes de:

- refactor amplio;
- experimento;
- migración;
- cambio de estrategia.

No sustituyen Git. Haz commits/checkpoints del repositorio apropiados para cambios importantes.

## Plan mode

Úsalo para investigación multiarchivo o diseño incierto. Omítelo cuando el diff es trivial. Un buen plan enumera archivos, contratos, pruebas, riesgos y acciones que requieren permiso.

## Auto mode

Si está disponible, aplica safety checks de fondo. Sigue necesitando scope, tests y revisión. No uses en repos/entornos desconocidos con credenciales amplias.

## Tool selection

- Connector/MCP antes que computer use para precisión.
- Search antes de leer cientos de archivos.
- Subagent para investigación que contaminaría contexto.
- Hook para regla determinista.
- Skill para procedimiento razonado.

## Auditoría

```text
/permissions → revisar reglas
/mcp → conexiones y costo/contexto
/context → qué consume contexto
/doctor → problemas de instalación/configuración
```

Confirma comandos disponibles en tu versión.

Fuentes: [tools reference](https://code.claude.com/docs/en/tools-reference), [permissions](https://code.claude.com/docs/en/permissions) y [cómo funciona Claude Code](https://code.claude.com/docs/en/how-claude-code-works).

