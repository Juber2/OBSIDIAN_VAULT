---
tags: [Claude, Modelos, ExtendedThinking, Research]
Creado: 2026-08-24
Actualizado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Modelos, extended thinking y modos

## Cuatro decisiones distintas

- Modelo disponible.
- Superficie (chat/Cowork/Code).
- Extended thinking/esfuerzo.
- Herramienta de información (web search/Research).

## Web search vs thinking vs Research

| Capacidad | Mejor para | Ejemplo de programación |
|---|---|---|
| Web search | Dato actual y acotado | API vigente de Angular |
| Extended thinking | Razonamiento difícil sin actualidad | depurar algoritmo/concurrencia |
| Research | Varias búsquedas/fuentes y síntesis | plan de migración tecnológica |
| Research + thinking | Investigación + decisión compleja | evaluar arquitectura/proveedor |

Extended thinking no consulta automáticamente la web. Research necesita web search habilitado y puede incorporar conectores internos.

## Elegir modelo

Evita fijar recomendaciones eternas por nombre. Evalúa las familias disponibles según:

- velocidad;
- calidad de razonamiento/código;
- contexto;
- tool use;
- consumo del plan;
- necesidad de profundidad.

Un modelo rápido sirve para transformaciones focalizadas. Un modelo más capaz/esfuerzo alto puede justificar su costo en debugging difícil, arquitectura y cambios largos.

## Cómo probar

Usa 5–10 tareas reales con mismo contexto:

- test que pasa/falla;
- criterios cumplidos;
- correcciones humanas;
- consumo;
- latencia;
- cambios fuera de alcance.

## No sobreusar thinking

Más razonamiento puede aumentar tiempo/consumo y exploración. Para rename, boilerplate o formato, usa respuesta directa. Para un problema difícil, activa thinking y proporciona criterios de verificación.

## Prompt útil

```text
Usa extended thinking para evaluar estas tres hipótesis del deadlock, pero no busques
la web. Apóyate solo en el código/logs adjuntos. Termina con el experimento mínimo
que falsaría cada hipótesis.
```

Fuente: [cuándo usar web search, extended thinking y Research](https://support.claude.com/en/articles/11095361-when-should-i-use-web-search-extended-thinking-and-research).

