---
tags: [Claude, CodeExecution, Files, DataAnalysis]
Creado: 2026-08-24
Relacionado: [CLAUDE PARA PROGRAMAR]
---

# Ejecución de código, archivos y datos

Claude chat puede ejecutar código en un entorno aislado y crear/editar archivos cuando la capacidad está habilitada.

## Capacidades

- ejecutar Python/JavaScript y utilidades disponibles;
- analizar CSV/TSV/PDF y otros archivos;
- crear visualizaciones PNG;
- producir `.xlsx`, `.pptx`, `.docx` y PDF;
- transformar formatos;
- trabajar con varios archivos;
- instalar paquetes desde fuentes permitidas si la red está habilitada.

Esto no es Claude Code: se ejecuta en un contenedor de conversación, no directamente en tu repo local.

## Programación

- reproducir un algoritmo mínimo;
- analizar logs y métricas;
- transformar datos de prueba;
- generar fixtures;
- convertir OpenAPI/CSV en documentación;
- crear reporte de performance;
- inspeccionar PDFs técnicos.

## Seguridad de red

El contenedor está aislado, pero con network egress puede descargar paquetes o acceder a dominios autorizados. Desactivar red reduce riesgo de exfiltración por prompt injection.

Para datos sensibles:

- elimina PII/secrets;
- usa red apagada cuando sea posible;
- verifica configuración de organización;
- revisa paquetes instalados;
- descarga e inspecciona resultados.

## Ejemplo

```text
Analiza estos logs CSV. No accedas a internet ni alteres el archivo original.
Genera: distribución de latencia, p95/p99 por endpoint, errores por ventana de 5 min,
anomalías y un notebook/script reproducible. Explica timezone y nulos.
```

## No confundir

| Necesidad | Herramienta |
|---|---|
| Snippet/datos/archivo generado | Code execution en chat |
| Prototipo interactivo | Artifact |
| Repo real + tests/build | Claude Code |
| Apps/archivos locales | Cowork/computer use |

## Verificación

- Revisa fórmulas y totales.
- Reejecuta scripts críticos localmente.
- Comprueba que el output abra/renderice.
- Conserva script y supuestos.
- No asumas que el entorno coincide con producción.

Fuente: [crear y editar archivos con Claude](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude).

