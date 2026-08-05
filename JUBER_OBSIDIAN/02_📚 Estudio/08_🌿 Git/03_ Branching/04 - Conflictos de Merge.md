---
tags:
  - Git
  - Conflictos
Creado: 2026-05-22
Relacionado:
  - Branching Git
---

# Conflictos de Merge

Un conflicto ocurre cuando Git no puede decidir cómo combinar cambios.

```text
<<<<<<< HEAD
cambio actual
=======
cambio entrante
>>>>>>> feature
```

## Proceso

1. Lee ambos lados.
2. Decide resultado correcto.
3. Elimina marcadores.
4. Corre tests.
5. `git add`.
6. Continua merge/rebase.

## Checklist

- [ ] No acepto cambios a ciegas.
- [ ] Entiendo ambos lados.
- [ ] Corro tests.
- [ ] Reviso diff final.
