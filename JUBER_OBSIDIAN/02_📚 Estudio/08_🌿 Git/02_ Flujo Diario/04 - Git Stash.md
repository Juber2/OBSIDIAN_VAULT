---
tags:
  - Git
  - Stash
Creado: 2026-05-22
Relacionado:
  - Flujo Diario Git
---

# Git Stash

`stash` guarda cambios temporales sin commitearlos.

```bash
git stash push -m "wip login"
git stash list
git stash pop
git stash apply stash@{0}
```

## Uso

- Cambiar de rama rápido.
- Guardar WIP temporal.
- Probar algo sin commit.

## Checklist

- [ ] Uso mensajes en stash.
- [ ] No dejo stashes olvidados.
- [ ] Entiendo `pop` vs `apply`.
- [ ] No uso stash como reemplazo de commits.
