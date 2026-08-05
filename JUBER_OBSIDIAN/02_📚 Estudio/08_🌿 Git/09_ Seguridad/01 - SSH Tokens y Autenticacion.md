---
tags:
  - GitHub
  - SSH
  - Seguridad
Creado: 2026-05-22
Relacionado:
  - Seguridad GitHub
---

# SSH, Tokens y Autenticación

GitHub permite autenticarse con SSH keys o tokens.

```bash
ssh-keygen -t ed25519 -C "tu@email.com"
ssh -T git@github.com
```

## Reglas

- No compartas llaves privadas.
- Usa passphrase.
- Revoca tokens que ya no uses.
- Usa permisos mínimos en tokens.

## Checklist

- [ ] SSH configurado.
- [ ] Tokens con permisos mínimos.
- [ ] Tokens expirables.
- [ ] Llaves antiguas removidas.
