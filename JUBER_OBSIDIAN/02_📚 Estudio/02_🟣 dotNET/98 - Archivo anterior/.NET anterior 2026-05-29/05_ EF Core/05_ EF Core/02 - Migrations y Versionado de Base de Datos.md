---
tags:
  - EFCore
  - Migrations
Creado: 2026-05-21
Relacionado:
  - Entity Framework Core
---

# Migrations y Versionado de Base de Datos

Migrations versiona cambios del modelo hacia la base.

```bash
dotnet ef migrations add CreateProducts
dotnet ef database update
```

## Reglas

- Revisa el SQL generado.
- No edites a ciegas migraciones ya aplicadas.
- Ten estrategia para datos existentes.
- En producción, aplica migraciones con pipeline o script controlado.

## Checklist

- [ ] Cada cambio de esquema tiene migration.
- [ ] Reviso impacto antes de aplicar.
- [ ] Evito cambios destructivos sin respaldo.
- [ ] No uso migraciones automáticas sin control en prod.
