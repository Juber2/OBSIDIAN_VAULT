---
tags:
  - GitHubActions
  - Seguridad
Creado: 2026-05-22
Actualizado: 2026-08-09
Relacionado:
  - GitHub Actions CI CD
---

# Secrets, Variables y Environments

Secrets guardan valores sensibles. Variables guardan configuración no secreta.

## Usos

| Tipo | Uso | Dónde se define |
|---|---|---|
| Secret | tokens, passwords, connection strings, keys | Settings → Secrets and variables → Actions |
| Variable | nombres, flags, URLs públicas | Settings → Secrets and variables → Actions (tab Variables) |
| Environment | dev, staging, prod, cada uno con sus propios secrets/variables y reglas | Settings → Environments |

```yaml
env:
  API_URL: ${{ vars.API_URL }}

steps:
  - run: echo "Deploying with token"
    env:
      DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

## Environments con protección

```yaml
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://miapp.com
    steps:
      - run: echo "deploy a prod"
```

En Settings → Environments → `production` se puede configurar:
- **Required reviewers**: alguien debe aprobar manualmente antes de que el job corra.
- **Wait timer**: espera N minutos antes de ejecutar (ventana para cancelar).
- **Deployment branches**: solo `main` (o tags) puede desplegar a ese environment.

## OIDC: evitar secrets de larga duración en la nube

En vez de guardar `AWS_SECRET_ACCESS_KEY` o credenciales de Azure como secret fijo, GitHub soporta **OpenID Connect (OIDC)**: el workflow pide un token de corta duración al proveedor cloud en el momento, sin secretos guardados.

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: azure/login@v2
    with:
      client-id: ${{ vars.AZURE_CLIENT_ID }}
      tenant-id: ${{ vars.AZURE_TENANT_ID }}
      subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
```

Esto es la práctica recomendada actual para Azure/AWS/GCP: menos superficie de ataque, sin rotar secretos manualmente.

## Buenas prácticas de seguridad

- Los secrets **nunca** se imprimen en logs (GitHub los enmascara automáticamente si coinciden exactamente), pero evitá loguearlos igual (ej. `echo $SECRET` en un script que los transforma).
- No uses secrets de `pull_request` en workflows que corren sobre PRs de forks externos sin revisión — un PR malicioso podría intentar exfiltrarlos. Para eso existe `pull_request_target` (usarlo con cuidado) o requerir aprobación manual para correr workflows de forks.
- Rotá secretos comprometidos inmediatamente y revisá el historial de Actions por si se filtraron en logs.
- Usá el permiso mínimo necesario con `permissions:` a nivel workflow/job en vez de dejar el default amplio.

```yaml
permissions:
  contents: read
  pull-requests: write   # solo si el job comenta en PRs
```

## Checklist
- [ ] No escribo secretos en YAML.
- [ ] Uso environments para prod con reviewers/approvals.
- [ ] Evalué usar OIDC en vez de secrets estáticos para cloud.
- [ ] Declaro `permissions:` mínimos por workflow/job.
- [ ] Roto secretos comprometidos y reviso logs históricos.
