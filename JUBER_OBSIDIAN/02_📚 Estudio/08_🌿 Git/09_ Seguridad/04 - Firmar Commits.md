---
tags:
  - Git
  - Seguridad
Creado: 2026-05-22
Relacionado:
  - Seguridad GitHub
---

# Firmar Commits

Firmar commits ayuda a verificar identidad del autor.

## Opciones

- GPG.
- SSH signing.
- S/MIME.

```bash
git config --global commit.gpgsign true
```

## Checklist

- [ ] Entiendo Verified commits.
- [ ] Configuro firma si el equipo lo exige.
- [ ] Protejo llave privada.
- [ ] No confundo firma con code review.
