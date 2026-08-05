---
tags:
  - GitHub
  - Merge
Creado: 2026-05-22
Relacionado:
  - Pull Requests
---

# Merge, Squash y Rebase Merge

GitHub permite varias estrategias al cerrar PR.

| Estrategia | Resultado |
|---|---|
| Merge commit | conserva commits y crea merge commit |
| Squash | combina commits en uno |
| Rebase merge | reaplica commits sin merge commit |

## Criterio

Squash funciona bien cuando los commits del PR son WIP. Merge commit sirve si quieres conservar contexto de rama.

## Checklist

- [ ] Entiendo política del repo.
- [ ] Squash si commits no aportan.
- [ ] Merge commit si importa historial de rama.
- [ ] Mensaje final claro.
