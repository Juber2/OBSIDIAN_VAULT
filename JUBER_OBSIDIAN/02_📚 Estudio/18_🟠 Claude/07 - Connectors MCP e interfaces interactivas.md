---
tags: [Claude, Connectors, MCP, InteractiveConnectors]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Connectors, MCP e interfaces interactivas

## Connectors

Integraciones con servicios para aportar datos o acciones. Programación:

- GitHub: código/issues/PRs;
- Google Drive/Docs: requisitos y documentación;
- Slack: contexto de incidentes/tareas;
- Asana/Linear: estado y tickets;
- Figma u otras herramientas: diseño/contexto.

## MCP

Protocolo para exponer tools/datos a Claude. Un servidor puede consultar DB, controlar navegador, leer logs o llamar una API interna.

MCP no enseña automáticamente el proceso de tu equipo; combínalo con una Skill.

```text
Connector/MCP = acceso
Skill = procedimiento/conocimiento
Permission = límite
Hook = control determinista (Claude Code)
```

## Interactive connectors

Algunos renderizan apps interactivas dentro del chat: tableros, dashboards o herramientas. Claude también puede producir visuales cuando no existe una interfaz adecuada.

Ventaja: interactuar con el sistema sin salir de la conversación. Riesgo: una interfaz cómoda sigue teniendo permisos reales.

## Elegir API, connector o computer use

| Opción | Precisión | Cuándo |
|---|---|---|
| Connector/MCP | Alta/estructurada | Integración disponible |
| Web | Lectura pública actual | Información |
| Computer use | Visual, más frágil | No hay API/conector |

## Programación: flujo de issue a PR

```text
Linear/Slack connector → requisito/contexto
GitHub connector → repo/issue/diff
Project → arquitectura
Claude Code → implementación/tests
Interactive connector → actualizar ticket, solo con aprobación
```

## Evaluar un conector/MCP

- proveedor y código;
- scopes OAuth;
- lectura/escritura;
- datos almacenados;
- separación de tenants;
- aprobación por acción;
- auditoría;
- revocación;
- exposición a prompt injection;
- comportamiento al desconectar.

## Higiene

Conecta solo lo que usarás. Nombres/tools innecesarios consumen atención/contexto y amplían superficie de riesgo. En Claude Code, usa `/mcp` para revisar conexiones/costos según documentación actual.

Fuentes: [interactive connectors](https://support.claude.com/en/articles/13454812-use-interactive-connectors-in-claude), [directorio unificado](https://support.claude.com/en/articles/14328846-browse-skills-connectors-and-plugins-in-one-directory) y [MCP en Claude Code](https://code.claude.com/docs/en/mcp).

