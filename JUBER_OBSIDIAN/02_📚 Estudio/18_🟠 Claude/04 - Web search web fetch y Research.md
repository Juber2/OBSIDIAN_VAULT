---
tags: [Claude, WebSearch, WebFetch, Research]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Web search, web fetch y Research

## Web search

Consulta información actual y devuelve fuentes. Programación:

- docs de framework/versión;
- CVE/advisory;
- changelog/deprecación;
- compatibilidad de paquetes;
- estándar o especificación.

## Web fetch

Con web search activo, Claude puede recuperar contenido de una URL directa. Úsalo para una página específica, release note o documento. Las páginas extensas consumen más contexto/uso.

## Research

Realiza varias búsquedas que se alimentan entre sí y sintetiza un informe con citas. Puede combinar web y conectores internos.

Casos:

- migración .NET/Angular mayor;
- comparación de bases de datos/cloud;
- auditoría pública de dependencias;
- investigación de arquitectura;
- actualización de documentación interna.

## Receta

```text
1. Web search: localizar documentación oficial.
2. Web fetch: leer páginas exactas.
3. Research: comparar y sintetizar si hay muchas fuentes.
4. Project: contrastar con arquitectura interna.
5. Claude Code: aplicar cambio y probar.
```

## Prompt

```text
Usa Research para preparar una migración de Angular [A] a [B]. Prioriza docs,
changelog y repos oficiales. Separa breaking changes confirmados, impacto inferido
en nuestros archivos del Project y pruebas necesarias. Cita versión/fecha.
```

## Límites

- Research consume más cuota.
- Web fetch largo puede llenar contexto.
- Una cita no valida todo el párrafo.
- Fuentes secundarias pueden estar desactualizadas.
- Contenido web puede intentar prompt injection.

Fuentes: [web search](https://support.claude.com/en/articles/10684626-enable-and-use-web-search), [Research](https://support.claude.com/en/articles/11088861-use-research-on-claude) y [cuándo usar cada capacidad](https://support.claude.com/en/articles/11095361-when-should-i-use-web-search-extended-thinking-and-research).

