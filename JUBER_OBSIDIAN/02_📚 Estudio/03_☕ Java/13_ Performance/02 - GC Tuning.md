---
tags:
  - Java
  - JVM
Creado: 2026-05-21
Relacionado:
  - Performance Java
---

# GC Tuning

El Garbage Collector libera memoria de objetos no referenciados. Tuning no significa tocar flags al azar, sino ajustar segun objetivos.

## GCs comunes

| GC | Uso |
|---|---|
| G1 | Default general, buen balance |
| ZGC | Latencia baja con heaps grandes |
| Shenandoah | Latencia baja |
| Parallel | Throughput alto, pausas mayores |

## Flags utiles

```bash
-Xms512m
-Xmx512m
-XX:+UseG1GC
-Xlog:gc*:file=gc.log:time,uptime,level,tags
```

## Senales

- Muchas pausas pequenas pueden indicar demasiadas asignaciones.
- Pausas largas pueden indicar heap grande, old gen presionado o full GC.
- Heap pequeno causa GC frecuente.
- Heap enorme puede aumentar pausas si el GC no es adecuado.

## Checklist

- [ ] Conozco objetivo: latencia o throughput.
- [ ] Reviso logs de GC.
- [ ] Ajusto memoria del contenedor.
- [ ] No cambio flags sin medir.
