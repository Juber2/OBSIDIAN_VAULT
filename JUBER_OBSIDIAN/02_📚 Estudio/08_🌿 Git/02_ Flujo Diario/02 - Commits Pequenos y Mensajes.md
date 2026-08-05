---
tags:
  - Git
  - Commits
Creado: 2026-05-22
Relacionado:
  - Flujo Diario Git
---

# Commits Pequeños y Mensajes

Un commit debe representar una unidad lógica de cambio.

## Buenas prácticas

- No mezclar refactor con feature si no hace falta.
- No commitear archivos generados por accidente.
- Mensaje en imperativo o intención clara.
- Un commit debe poder revisarse.

```bash
git commit -m "Fix order total calculation"
```

## Checklist

- [ ] Commit pequeño.
- [ ] Mensaje claro.
- [ ] Sin cambios accidentales.
- [ ] Compila o al menos no rompe conscientemente.
