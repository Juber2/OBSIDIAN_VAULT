---
tags:
  - GitHubActions
  - Deploy
Creado: 2026-05-22
Relacionado:
  - GitHub Actions CI CD
---

# Deploy con GitHub Actions

Un workflow de deploy debe usar artefactos probados, environments y secretos seguros.

## Flujo recomendado

```text
PR -> CI -> merge main -> build artifact -> deploy staging/prod
```

## Reglas

- No despliegues si tests fallan.
- Usa environments para producción.
- Separa build de deploy si el proyecto crece.
- Guarda evidencia en logs.

## Checklist

- [ ] Deploy depende de CI exitoso.
- [ ] Secrets protegidos.
- [ ] Producción tiene aprobación si aplica.
- [ ] Puedo hacer rollback.
