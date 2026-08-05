---
tags:
  - NET
  - Configuracion
Creado: 2026-05-21
Relacionado:
  - DevOps con .NET
---

# Configuración por Ambientes

.NET usa `appsettings.json`, variables de entorno y providers externos.

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=App;"
  }
}
```

## Orden común

- `appsettings.json`
- `appsettings.Development.json`
- variables de entorno
- Azure Key Vault u otro secret manager

## Checklist

- [ ] No guardo secretos en repo.
- [ ] Uso options tipadas.
- [ ] Valido configuración al iniciar.
- [ ] Separo local/dev/prod.
