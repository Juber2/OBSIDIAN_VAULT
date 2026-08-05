---
tags:
  - NET
  - GC
Creado: 2026-05-21
Relacionado:
  - Performance .NET
---

# GC, Memoria y Allocations

El GC limpia objetos no referenciados, pero demasiadas asignaciones generan presión.

## Generaciones

| Generación | Tipo de objetos |
|---|---|
| Gen 0 | objetos recientes |
| Gen 1 | transición |
| Gen 2 | objetos de vida larga |
| LOH | objetos grandes |

## Checklist

- [ ] Detecto allocations excesivas.
- [ ] Evito strings temporales en loops críticos.
- [ ] Uso pooling cuando hay beneficio claro.
- [ ] Reviso LOH si hay objetos grandes.
