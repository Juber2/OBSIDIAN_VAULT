---
tags:
  - dotnet
  - senior
  - incidentes
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Referencia Senior dotNET
  - NET
---

# Playbook de Incidentes dotNET

## Primeros 10 minutos

Confirmar impacto, mirar health checks, errores recientes, latencia, cambios desplegados y dependencias externas.

## Datos a revisar

Logs por correlation id, traces lentos, metricas de CPU/memoria/GC, conexiones DB, cola de mensajes, excepciones top.

## Mitigacion

Rollback, feature flag, escalar recursos, pausar consumidor, limpiar cola o desactivar integracion externa segun caso.

## Postmortem

Linea de tiempo, causa raiz, impacto, deteccion, respuesta, acciones preventivas y dueño. Sin culpas personales.

## Relacionado

- [[Referencia Senior dotNET]]
- [[NET]]
