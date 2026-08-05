---
tags:
  - NET
  - TDD
Creado: 2026-05-21
Relacionado:
  - Testing en .NET
---

# TDD y Test Design

TDD propone ciclos cortos: red, green, refactor.

## Ciclo

1. Escribe un test que falla.
2. Implementa lo mínimo para pasar.
3. Refactoriza manteniendo tests verdes.

## Buen test

- Explica comportamiento.
- Es independiente.
- Tiene datos claros.
- Falla por una razón útil.

## Checklist

- [ ] Testeo comportamiento, no implementación.
- [ ] Mantengo fixtures simples.
- [ ] Evito sleeps y dependencia de tiempo real.
- [ ] Refactorizo con tests como red de seguridad.
