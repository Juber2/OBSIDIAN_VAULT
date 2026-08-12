---
tags:
  - GitHubActions
  - Deploy
Creado: 2026-05-22
Actualizado: 2026-08-09
Relacionado:
  - GitHub Actions CI CD
---

# Deploy con GitHub Actions

Un workflow de deploy debe usar artefactos probados, environments y secretos seguros.

## Flujo recomendado

```text
PR -> CI -> merge main -> build artifact -> deploy staging -> (aprobación) -> deploy prod
```

## Separar build de deploy

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: dotnet publish -c Release -o out
      - uses: actions/upload-artifact@v4
        with:
          name: app
          path: out

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app
      - run: echo "desplegando a staging"

  deploy-prod:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://miapp.com
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app
      - run: echo "desplegando a producción"
```

`deploy-prod` con `environment: production` respeta los **required reviewers** configurados — el pipeline se pausa hasta que alguien aprueba.

## Deploy a Azure (ejemplo con Web App)

```yaml
- uses: azure/login@v2
  with:
    client-id: ${{ vars.AZURE_CLIENT_ID }}
    tenant-id: ${{ vars.AZURE_TENANT_ID }}
    subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}

- uses: azure/webapps-deploy@v3
  with:
    app-name: mi-app-prod
    package: out
```

## Deploy con Docker

```yaml
- uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

- uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
```

## Estrategias de deploy

| Estrategia | Idea | Cuándo usarla |
|---|---|---|
| Rolling | reemplaza instancias de a poco | default en la mayoría de PaaS |
| Blue-Green | dos ambientes idénticos, se cambia el router de uno a otro | rollback instantáneo, requiere doble infra |
| Canary | despliega a un % chico de tráfico primero | apps con mucho tráfico, reduce blast radius |

## Rollback

- Si el deploy usa artifact versionado (tag/SHA), rollback = volver a desplegar el artifact anterior, no revertir código y re-buildear.
- Guardar el `github.sha` desplegado como referencia (ej. en un archivo `VERSION` o tag de Docker) para saber exactamente qué está corriendo en cada ambiente.

```yaml
- run: echo "Deployed SHA: ${{ github.sha }}" >> deploy-log.txt
```

## Self-hosted runners para deploys internos

Si el deploy necesita acceso a una red privada (ej. servidor on-prem sin exponer a internet), un runner self-hosted dentro de esa red resuelve el problema sin abrir el firewall.

```yaml
jobs:
  deploy:
    runs-on: self-hosted
    steps:
      - run: ./deploy.sh
```

## Checklist
- [ ] Deploy depende de CI exitoso.
- [ ] Build y deploy están separados (artifact reutilizado, no rebuild).
- [ ] Secrets protegidos, idealmente vía OIDC.
- [ ] Producción tiene aprobación (required reviewers).
- [ ] Puedo hacer rollback rápido a una versión anterior conocida.
- [ ] Elegí estrategia de deploy acorde al riesgo/tráfico de la app.
