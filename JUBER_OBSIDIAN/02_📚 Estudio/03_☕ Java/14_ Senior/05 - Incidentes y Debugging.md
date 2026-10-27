---
tags:
  - Java
  - Debugging
Creado: 2026-05-21
Relacionado:
  - Senior Java Engineer
---

# Incidentes y Debugging

Debugging senior es reducir incertidumbre bajo presion.

## Secuencia

1. Define impacto: usuarios, endpoints, ambiente.
2. Revisa cambios recientes.
3. Mira logs, metricas y traces.
4. Formula hipotesis.
5. Prueba una hipotesis a la vez.
6. Mitiga primero, arregla despues.
7. Escribe postmortem si fue relevante.

## Herramientas Java

- Logs con traceId.
- Actuator health/metrics.
- Thread dumps.
- Heap dumps.
- GC logs.
- APM.

## Preguntas utiles

- Empezo despues de deploy?
- Afecta a todos o a un segmento?
- Hay aumento de latencia o errores?
- La base de datos esta lenta?
- Hay saturacion de pool?

## Checklist

- [ ] No reinicio a ciegas sin capturar evidencia si puedo evitarlo.
- [ ] Tengo rollback o feature flag.
- [ ] Comunico estado con claridad.
- [ ] Documento causa raiz y acciones preventivas.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

