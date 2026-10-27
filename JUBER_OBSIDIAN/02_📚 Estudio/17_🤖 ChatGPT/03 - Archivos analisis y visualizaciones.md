---
tags: [ChatGPT, Archivos, Analisis, Visualizaciones]
Creado: 2026-08-24
Relacionado: [CHATGPT]
---

# Archivos, análisis y visualizaciones

## Trabajar con archivos

ChatGPT puede usar archivos como código, documentos, PDFs, hojas de cálculo, imágenes y exports. La capacidad exacta depende de superficie y plan.

### Usos al programar

- analizar logs CSV/JSON;
- comparar configuraciones;
- revisar un PDF de arquitectura;
- convertir OpenAPI en documentación o casos de prueba;
- analizar métricas y perfiles;
- crear una matriz de dependencias;
- generar documentos o reportes editables.

## Cómo preparar archivos

1. Adjunta solo fuentes necesarias.
2. Mantén nombre y ruta relativa.
3. Indica versión/commit/fecha.
4. Explica la autoridad de cada archivo.
5. Elimina secrets y PII.
6. Pide que cite archivo y símbolo/hoja/celda.

## Análisis de datos

Para logs o rendimiento, define:

- pregunta de negocio/técnica;
- columnas y unidades;
- período/zona horaria;
- significado de nulos;
- baseline y umbral;
- formato de salida.

Pide conservar el archivo original y generar un resultado nuevo, reproducible y con supuestos.

## Visualizations

Crean charts, mapas, diagramas, calculadoras, simulaciones y explicaciones interactivas. Son útiles cuando cambiar parámetros o ver relaciones mejora la comprensión.

### Aplicaciones de programación

- grafo de módulos/dependencias;
- secuencia de autenticación;
- calculadora de capacidad;
- simulador de retries/backoff;
- diagrama de estados;
- dashboard de logs;
- comparación de planes de ejecución.

### Prompt ejemplo

```text
Crea una visualización interactiva del flujo OAuth: navegador, frontend, backend,
proveedor y base de datos. Permite alternar entre éxito, token vencido y state inválido.
Cada paso debe mostrar datos enviados, frontera de confianza y posible error.
```

## Visualización ≠ documentación canónica

Exporta o replica la decisión en Mermaid/ADR/código versionado. Una visualización dentro de un chat puede quedar desactualizada.

## Verificación

- Compara totales con fuente.
- Revisa fórmulas y filtros.
- Confirma unidades/zonas.
- Inspecciona archivos creados.
- Reproduce análisis crítico localmente.

Fuentes: [trabajar con archivos](https://learn.chatgpt.com/docs/artifacts-viewer) y [Visualizations](https://learn.chatgpt.com/docs/visualizations).
