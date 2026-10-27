---
tags:
  - Java
  - Performance
Creado: 2026-05-21
Relacionado:
  - Performance Java
---

# Profiling y Diagnostico

Profiling es observar donde consume CPU, memoria, IO o tiempo una aplicacion.

## Herramientas

| Herramienta | Uso |
|---|---|
| Java Flight Recorder | Perfil de bajo overhead |
| Mission Control | Analizar grabaciones JFR |
| VisualVM | Inspeccion local |
| async-profiler | CPU y alloc profiling avanzado |
| jstack | Thread dumps |
| jmap | Heap dumps |

## Sintomas y pistas

| Sintoma | Posibles causas |
|---|---|
| CPU alta | loops, serializacion, regex, GC |
| Latencia alta | DB lenta, locks, llamadas externas |
| Memoria crece | leaks, caches sin limite, listas grandes |
| Threads bloqueados | locks, pool pequeno, IO lento |

## Checklist

- [ ] Tengo una medicion base.
- [ ] Identifique cuello de botella real.
- [ ] Diferencio CPU, memoria, DB y red.
- [ ] Guardo evidencia antes y despues del cambio.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

