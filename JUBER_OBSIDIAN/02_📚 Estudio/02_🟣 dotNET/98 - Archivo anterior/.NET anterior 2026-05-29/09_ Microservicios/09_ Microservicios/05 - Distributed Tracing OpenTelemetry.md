---
tags:
  - NET
  - OpenTelemetry
Creado: 2026-05-21
Relacionado:
  - Microservicios .NET
---

# Distributed Tracing y OpenTelemetry

Tracing distribuido muestra el recorrido de una request entre servicios.

## OpenTelemetry

Permite capturar traces, métricas y logs de forma estándar.

## Qué mirar

- TraceId.
- Spans por servicio.
- Latencia.
- Errores.
- Dependencias lentas.

## Checklist

- [ ] Propago traceId.
- [ ] Instrumento HTTP y DB.
- [ ] Puedo seguir una request completa.
- [ ] Uso dashboards para incidentes.
