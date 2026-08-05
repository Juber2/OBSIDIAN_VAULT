---
tags:
  - Git
  - Recuperacion
Creado: 2026-05-22
Relacionado:
  - Flujo Diario Git
---

# Restore, Reset y Revert

Comandos para deshacer, con impactos distintos.

| Comando | Uso |
|---|---|
| `restore` | descartar o sacar del staging |
| `reset` | mover HEAD/rama |
| `revert` | crear commit que deshace otro |

```bash
git restore file.txt
git restore --staged file.txt
git revert abc123
```

## Regla senior

En ramas compartidas, prefiere `revert` antes que reescribir historia.

## Checklist

- [ ] Sé descartar cambios locales.
- [ ] Sé sacar del staging.
- [ ] Uso revert en historial compartido.
- [ ] Tengo cuidado con reset hard.
