---
tags:
  - Java
  - CodeReview
Creado: 2026-05-21
Relacionado:
  - Senior Java Engineer
---

# Code Review Senior

Un buen review no busca demostrar que sabes mas. Busca reducir riesgos y mejorar el codigo del equipo.

## Que revisar

| Area | Preguntas |
|---|---|
| Correctitud | Cumple el caso? rompe otros flujos? |
| Tests | Hay pruebas utiles? cubren borde? |
| Seguridad | Expone datos? valida permisos? |
| Performance | Hay N+1, loops caros, payloads enormes? |
| Mantenibilidad | Nombres claros? responsabilidades separadas? |

## Comentario util

```text
Este metodo mezcla validacion, persistencia y envio de eventos. Podriamos mover la regla de validacion al caso de uso para testearla sin Spring y dejar el adapter solo para infraestructura.
```

## Checklist

- [ ] Priorizo bugs y riesgos sobre gustos.
- [ ] Propongo alternativas concretas.
- [ ] Diferencio bloqueante de sugerencia.
- [ ] Reviso tests con la misma seriedad que codigo productivo.
