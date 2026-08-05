---
tags:
  - Git
  - Worktree
Creado: 2026-05-22
Relacionado:
  - Git Avanzado
---

# Git Worktree

`git worktree` permite tener varias ramas en carpetas distintas sin clonar de nuevo.

```bash
git worktree add ../repo-hotfix hotfix/payment
git worktree list
git worktree remove ../repo-hotfix
```

## Uso

- Hotfix urgente.
- Revisar otra rama.
- Trabajar en paralelo.

## Checklist

- [ ] Uso worktree para evitar stash innecesario.
- [ ] Limpio worktrees viejos.
- [ ] Entiendo que comparten repo interno.
- [ ] No confundo carpetas activas.
