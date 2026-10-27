---
tags:
  - Postulaciones
  - Entrevistas
  - Simulaciones
Creado: 2026-08-30
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Entrevistas técnicas y simulaciones

## Formato de respuesta técnica

Usar esta secuencia:

1. **Definición breve.**
2. **Caso de uso.**
3. **Ejemplo propio.**
4. **Trade-off o error común.**
5. **Cómo se verifica en producción.**

Ejemplo: “OpenTelemetry es un estándar de instrumentación para traces, metrics y logs. Lo usaría para correlacionar una petición desde la API hasta SQL o una cola. En mi proyecto… Su costo principal es el volumen y la cardinalidad, por eso aplicaría sampling y evitaría IDs de usuario como etiquetas de métricas.”

## Rondas semanales

### Ronda 1 — Fundamentos, 30 minutos

- 5 preguntas Angular.
- 5 preguntas C#/.NET.
- 3 preguntas SQL/HTTP.

### Ronda 2 — Implementación, 60-90 minutos

- Crear o corregir una feature pequeña.
- Pensar en voz alta.
- Escribir pruebas mínimas.
- Explicar complejidad y limitaciones.

### Ronda 3 — System design, 45 minutos

- Aclarar requisitos.
- Dibujar solución.
- Analizar datos, fallos, seguridad, observabilidad y despliegue.

### Ronda 4 — Conductual, 30 minutos

Preparar historias STAR sobre:

- incidente difícil;
- desacuerdo técnico;
- error propio;
- mejora de calidad;
- entrega con presión;
- aprendizaje rápido;
- colaboración con producto o QA.

## Live coding

Practicar:

- manipulación de colecciones y LINQ;
- asincronía y cancelación;
- endpoint CRUD;
- consulta SQL y optimización;
- componente/formulario Angular;
- RxJS para búsqueda o dependencias;
- refactor y debugging de código existente.

No memorizar algoritmos exóticos como prioridad para puestos empresariales, pero manejar arrays, strings, hash maps, complejidad Big O y problemas básicos.

## Feedback después de cada simulación

| Dimensión | 1-5 | Acción |
|---|---:|---|
| Exactitud técnica | | |
| Claridad | | |
| Ejemplos reales | | |
| Trade-offs | | |
| Velocidad | | |
| Comunicación | | |

## Material del vault

- [[Preguntas de Entrevista Angular]]
- [[Preguntas de Entrevista dotNET]]
- [[Preguntas Arquitectura Senior]]
- [[15 - Banco de preguntas Full Stack]]

## Criterio de salida

Responder durante 45 minutos sin convertir cada pregunta en una definición memorizada. El entrevistador debe percibir implementación, criterio y capacidad de diagnóstico.

