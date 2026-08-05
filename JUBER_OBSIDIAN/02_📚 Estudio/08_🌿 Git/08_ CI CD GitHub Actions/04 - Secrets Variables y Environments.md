---
tags:
  - GitHubActions
  - Seguridad
Creado: 2026-05-22
Relacionado:
  - GitHub Actions CI CD
---

# Secrets, Variables y Environments

Secrets guardan valores sensibles. Variables guardan configuración no secreta.

## Usos

| Tipo | Uso |
|---|---|
| Secret | tokens, passwords, keys |
| Variable | nombres, flags, URLs públicas |
| Environment | dev, staging, prod con reglas |

```yaml
env:
  API_URL: ${{ vars.API_URL }}
```

## Checklist

- [ ] No escribo secretos en YAML.
- [ ] Uso environments para prod.
- [ ] Protejo deploys con approvals si aplica.
- [ ] Roto secretos comprometidos.
