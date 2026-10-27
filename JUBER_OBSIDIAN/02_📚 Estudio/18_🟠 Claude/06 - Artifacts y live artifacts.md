---
tags: [Claude, Artifacts, LiveArtifacts, Prototipos]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Artifacts y live artifacts

## Qué es un Artifact

Contenido significativo y autocontenido en una ventana separada del chat, pensado para editar, iterar, reutilizar o compartir.

Tipos comunes:

- Markdown/documentos;
- código;
- HTML de una página;
- componentes React;
- SVG;
- Mermaid/diagramas;
- visualizaciones/herramientas interactivas.

## Cuándo usarlo al programar

- prototipo UI antes de Angular/React real;
- playground de algoritmo;
- diagrama navegable;
- configurador/calculadora;
- demo para stakeholders;
- walkthrough de PR;
- dashboard desde datos de sesión.

## Iteración

Pide cambios concretos sobre el Artifact, usa historial/versiones y prueba la interacción. Si aparece un error, la interfaz puede ofrecer corregirlo con Claude.

## AI-powered Artifacts

Pueden incorporar capacidades de Claude. Los usuarios usan su propia cuenta/límites según el producto. Diseña:

- inputs y output;
- límites/guardrails;
- datos almacenados;
- fallos;
- costos/uso;
- comportamiento sin IA.

## MCP y almacenamiento

Según plan/configuración, un Artifact puede conectarse a MCP y usar almacenamiento persistente. Revisa si los datos son personales o compartidos antes de ingresar información sensible.

## Publicar/compartir

La modalidad cambia por plan: publicación pública o sharing organizacional. Antes:

- revisa código y dependencias;
- elimina adjuntos/datos sensibles;
- confirma audiencia;
- prueba con cuenta sin privilegios;
- entiende persistencia y revocación;
- conserva fuente canónica.

## Live Artifacts

En Cowork pueden ser dashboards HTML persistentes que refrescan datos conectados. También hay Artifacts asociados a Claude Code en planes/superficies compatibles para walkthroughs, dashboards o timelines privados/organizacionales.

## Pasar a producción

```text
Artifact → validar idea/UX
→ exportar requisitos/código
→ Claude Code integra al repo/design system
→ tests/build/browser QA
→ deploy corporativo
```

No pegues un componente Artifact directamente sin adaptar framework, seguridad, accesibilidad, estado, tests y licencias.

Fuentes: [qué son los Artifacts](https://support.claude.com/en/articles/9487310-what-are-artifacts-and-how-do-i-use-them), [publicar/compartir](https://support.claude.com/en/articles/9547008-publish-and-share-artifacts) y [live artifacts](https://support.claude.com/en/articles/14729249-use-live-artifacts-in-claude-cowork).

