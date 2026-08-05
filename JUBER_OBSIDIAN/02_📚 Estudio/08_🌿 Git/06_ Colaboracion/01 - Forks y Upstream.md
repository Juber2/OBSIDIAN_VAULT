---
tags:
  - GitHub
  - Forks
Creado: 2026-05-22
Relacionado:
  - Colaboracion GitHub
---

# Forks y Upstream

Un fork es una copia de un repo en tu cuenta.

```bash
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git switch main
git merge upstream/main
```

## Uso

- Contribuir a open source.
- Trabajar sin permisos directos.

## Checklist

- [ ] Entiendo origin vs upstream.
- [ ] Mantengo fork actualizado.
- [ ] Hago PR hacia repo original.
- [ ] No mezclo ramas de trabajo.
