---
tags:
  - Git
  - Workflow
Creado: 2026-05-22
Relacionado:
  - Flujo Diario Git
---

# Status, Diff, Add y Restore

Flujo diario antes de commitear.

```bash
git status
git diff
git add src/app.ts
git diff --staged
git restore src/app.ts
git restore --staged src/app.ts
```

## Checklist

- [ ] Reviso archivos modificados.
- [ ] Reviso diff antes de staging.
- [ ] Reviso diff staged antes de commit.
- [ ] Sé sacar archivos del staging.
