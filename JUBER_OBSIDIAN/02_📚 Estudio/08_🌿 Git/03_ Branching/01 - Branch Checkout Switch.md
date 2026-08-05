---
tags:
  - Git
  - Branches
Creado: 2026-05-22
Relacionado:
  - Branching Git
---

# Branch, Checkout y Switch

Una rama es un puntero a un commit.

```bash
git branch
git switch -c feature/login
git switch main
```

## Naming

```text
feature/login
fix/order-total
chore/update-deps
hotfix/payment-error
```

## Checklist

- [ ] Creo ramas por tarea.
- [ ] Uso nombres descriptivos.
- [ ] No trabajo directo en main.
- [ ] Limpio ramas viejas.
