---
tags:
  - Git
  - Reflog
Creado: 2026-05-22
Relacionado:
  - Git Avanzado
---

# Reflog y Recuperar Cambios

`reflog` registra movimientos de HEAD.

```bash
git reflog
git switch -c recovery abc123
```

## Uso

- Recuperar commit perdido.
- Deshacer reset accidental.
- Encontrar estado anterior.

## Checklist

- [ ] Reviso reflog antes de desesperar.
- [ ] Creo rama de recuperación.
- [ ] No uso comandos destructivos sin backup.
- [ ] Entiendo que reflog es local.
