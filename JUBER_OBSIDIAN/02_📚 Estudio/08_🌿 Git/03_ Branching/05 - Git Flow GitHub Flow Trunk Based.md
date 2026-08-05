---
tags:
  - Git
  - Workflow
Creado: 2026-05-22
Relacionado:
  - Branching Git
---

# Git Flow, GitHub Flow y Trunk-Based

Estrategias de ramas para equipos.

| Estrategia | Uso |
|---|---|
| Git Flow | releases formales, ramas largas |
| GitHub Flow | main + feature branches + PR |
| Trunk-Based | commits frecuentes a trunk con CI fuerte |

## Recomendación práctica

Para muchos equipos modernos: GitHub Flow o trunk-based con feature flags.

## Checklist

- [ ] Elijo estrategia por contexto.
- [ ] Evito ramas largas sin necesidad.
- [ ] Main siempre estable.
- [ ] CI protege integración.
