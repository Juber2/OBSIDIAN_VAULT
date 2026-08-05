---
tags:
  - Git
  - Merge
Creado: 2026-05-22
Relacionado:
  - Branching Git
---

# Merge y Fast Forward

Merge integra cambios de una rama en otra.

```bash
git switch main
git merge feature/login
```

## Fast-forward

Ocurre cuando la rama destino puede avanzar directo al commit de la rama fuente.

## Merge commit

Si ambas ramas avanzaron, Git crea un commit de merge.

## Checklist

- [ ] Entiendo fast-forward.
- [ ] Sé cuándo aparece merge commit.
- [ ] Reviso conflictos.
- [ ] Corro tests después de merge.
