---
tags:
  - Git
  - GitHub
  - Remote
Creado: 2026-05-22
Relacionado:
  - GitHub Fundamentos
---

# Remote, Origin, Push, Pull y Fetch

Un remote es una URL donde vive una copia del repo.

```bash
git remote -v
git remote add origin https://github.com/user/repo.git
git push -u origin main
git fetch origin
git pull
```

## Fetch vs Pull

| Comando | Qué hace |
|---|---|
| `fetch` | descarga referencias sin mezclar |
| `pull` | fetch + merge/rebase |

## Checklist

- [ ] Entiendo `origin`.
- [ ] Sé empujar una rama.
- [ ] Uso fetch para revisar antes.
- [ ] Entiendo qué hace pull.
