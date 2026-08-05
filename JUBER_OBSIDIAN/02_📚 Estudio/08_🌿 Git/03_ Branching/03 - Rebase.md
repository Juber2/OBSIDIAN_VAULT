---
tags:
  - Git
  - Rebase
Creado: 2026-05-22
Relacionado:
  - Branching Git
---

# Rebase

Rebase reaplica commits encima de otra base.

```bash
git switch feature/login
git rebase main
```

## Uso

- Mantener historial lineal.
- Actualizar rama con main.
- Limpiar commits locales.

## Cuidado

No rebases commits que otros ya están usando, salvo acuerdo del equipo.

## Checklist

- [ ] Entiendo que reescribe historia.
- [ ] Lo uso en ramas personales.
- [ ] No reescribo ramas compartidas sin coordinación.
- [ ] Sé abortar: `git rebase --abort`.
