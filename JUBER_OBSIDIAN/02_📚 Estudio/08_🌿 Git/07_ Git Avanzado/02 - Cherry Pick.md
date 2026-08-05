---
tags:
  - Git
  - CherryPick
Creado: 2026-05-22
Relacionado:
  - Git Avanzado
---

# Cherry Pick

Cherry-pick aplica un commit específico en otra rama.

```bash
git cherry-pick abc123
```

## Uso

- Llevar hotfix a otra rama.
- Aplicar commit puntual.
- Recuperar cambio sin merge completo.

## Checklist

- [ ] El commit es independiente.
- [ ] Reviso conflictos.
- [ ] Corro tests.
- [ ] Evito duplicar cambios sin rastreo.
