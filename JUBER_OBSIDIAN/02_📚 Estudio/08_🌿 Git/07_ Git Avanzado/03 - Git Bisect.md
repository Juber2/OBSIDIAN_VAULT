---
tags:
  - Git
  - Debugging
Creado: 2026-05-22
Relacionado:
  - Git Avanzado
---

# Git Bisect

`git bisect` ayuda a encontrar el commit que introdujo un bug.

```bash
git bisect start
git bisect bad
git bisect good v1.0.0
```

Git va probando commits intermedios.

## Checklist

- [ ] Tengo un commit bueno conocido.
- [ ] Tengo un commit malo.
- [ ] Puedo ejecutar prueba.
- [ ] Termino con `git bisect reset`.
