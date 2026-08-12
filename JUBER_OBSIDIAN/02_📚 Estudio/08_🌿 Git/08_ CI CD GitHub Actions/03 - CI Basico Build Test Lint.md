---
tags:
  - GitHubActions
  - CI
Creado: 2026-05-22
Actualizado: 2026-08-09
Relacionado:
  - GitHub Actions CI CD
---

# CI Básico: Build, Test y Lint

Un CI mínimo valida que el proyecto compile y pase pruebas antes de mergear.

## Ejemplo Node/Angular

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: "npm"          # cachea node_modules automáticamente
- run: npm ci               # más rápido y reproducible que npm install en CI
- run: npm run lint
- run: npm test -- --watch=false
- run: npm run build
```

## Ejemplo .NET

```yaml
- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: 8.0.x
- run: dotnet restore
- run: dotnet build --no-restore --configuration Release
- run: dotnet test --no-build --configuration Release --logger trx --results-directory TestResults
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: test-results
    path: TestResults
```

## Workflow completo de PR con protección

```yaml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: 8.0.x
      - run: dotnet restore
      - run: dotnet build --no-restore
      - run: dotnet test --no-build
```

Para que esto realmente bloquee merges: en GitHub → Settings → Branches → Branch protection rule sobre `main` → marcar el job `ci` como **required status check**.

## Publicar cobertura de tests

```yaml
- run: dotnet test --collect:"XPlat Code Coverage"
- uses: codecov/codecov-action@v4
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
```

## Monorepo: correr CI solo si cambió esa app

```yaml
on:
  pull_request:
    paths:
      - "apps/api/**"

jobs:
  ci-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cd apps/api && npm ci && npm test
```

## Badge de estado en el README

```markdown
![CI](https://github.com/usuario/repo/actions/workflows/ci.yml/badge.svg)
```

## Checklist
- [ ] CI corre en PR.
- [ ] Tests fallidos bloquean merge (branch protection + required check).
- [ ] Build production pasa.
- [ ] Logs son fáciles de leer.
- [ ] Uso cache de dependencias para acelerar el pipeline.
- [ ] Subo resultados de test como artifact cuando falla.
- [ ] En monorepo, filtro por `paths` para no correr CI de más.
