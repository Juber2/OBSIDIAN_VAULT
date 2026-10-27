---
tags: [ChatGPT, WebSearch, Browser, Investigacion]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Web search, browser e investigación

## Web search

Para preguntas actuales y acotadas: documentación de una versión, changelog, CVE, compatibilidad, deprecación o configuración vigente.

```text
Busca documentación oficial de Angular [versión] sobre [tema]. No uses tutoriales
como fuente principal. Cita la sección exacta, fecha/versión y contrasta con este código.
```

## Browser

Permite navegar e inspeccionar páginas de forma interactiva. Para desarrollo:

- leer documentación con navegación dinámica;
- probar una aplicación local;
- inspeccionar estados visibles;
- capturar screenshots;
- reproducir un flujo en un sitio autorizado.

El browser observa la interfaz; no sustituye tests de integración ni acceso semántico a la base de datos.

## Investigación profunda/long-running work

Úsala para una pregunta que requiere varias fuentes y síntesis:

- migración entre versiones mayores;
- comparación de proveedores;
- análisis de vulnerabilidades/dependencias;
- diseño basado en especificaciones y benchmarks;
- investigación de una tecnología emergente.

Define antes:

- pregunta de decisión;
- fuentes permitidas;
- fecha/versiones;
- criterios de comparación;
- evidencia mínima;
- formato final.

## Jerarquía de fuentes técnicas

1. Especificación/estándar.
2. Documentación oficial.
3. Repositorio y changelog oficial.
4. Advisory primario.
5. Paper/dataset original.
6. Fuente secundaria solo para contexto.

## Combinación potente

```text
Web search → obtener docs actuales
Browser → verificar el flujo real/documentación interactiva
Files → contrastar con repo/logs
Codex → aplicar y probar el cambio
```

## Límites y seguridad

- Una cita puede apoyar solo parte de una conclusión.
- El contenido web puede contener prompt injection.
- No ingreses credenciales en sitios no autorizados.
- Confirma dominio y acción antes de enviar formularios.
- Usa fechas absolutas y versiones.
- Revisa fuentes tú mismo en decisiones críticas.

Fuentes: [web search](https://learn.chatgpt.com/docs/web-search), [browser](https://learn.chatgpt.com/docs/browser) y [trabajo prolongado](https://learn.chatgpt.com/docs/long-running-work).

