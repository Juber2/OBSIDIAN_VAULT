---
tags:
  - Git
  - Rebase
Creado: 2026-05-22
Relacionado:
  - Git Avanzado
---

# Interactive Rebase, Squash y Fixup

Interactive rebase permite limpiar commits locales.

```bash
git rebase -i HEAD~4
```

## Acciones

| Acción | Uso |
|---|---|
| pick | mantener |
| reword | cambiar mensaje |
| squash | combinar con anterior |
| fixup | combinar descartando mensaje |
| drop | eliminar |

## Checklist

- [ ] Solo en ramas personales.
- [ ] Limpio commits WIP.
- [ ] No reescribo historia compartida sin acuerdo.
- [ ] Reviso resultado final.
