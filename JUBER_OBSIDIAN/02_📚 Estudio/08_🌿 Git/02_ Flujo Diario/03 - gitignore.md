---
tags:
  - Git
  - gitignore
Creado: 2026-05-22
Relacionado:
  - Flujo Diario Git
---

# .gitignore

`.gitignore` evita versionar archivos que no deben ir al repo.

```gitignore
node_modules/
bin/
obj/
.env
dist/
```

## No ignorar

No ignores archivos necesarios para construir o entender el proyecto.

## Checklist

- [ ] Ignoro dependencias instaladas.
- [ ] Ignoro builds generados.
- [ ] Ignoro secretos locales.
- [ ] No ignoro archivos necesarios.
