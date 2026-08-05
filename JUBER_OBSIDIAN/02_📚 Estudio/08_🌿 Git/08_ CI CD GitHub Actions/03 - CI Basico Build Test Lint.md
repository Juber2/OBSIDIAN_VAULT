---
tags:
  - GitHubActions
  - CI
Creado: 2026-05-22
Relacionado:
  - GitHub Actions CI CD
---

# CI Básico: Build, Test y Lint

Un CI mínimo valida que el proyecto compile y pase pruebas.

## Ejemplo Node/Angular

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
- run: npm ci
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
- run: dotnet build --no-restore
- run: dotnet test --no-build
```

## Checklist

- [ ] CI corre en PR.
- [ ] Tests fallidos bloquean merge.
- [ ] Build production pasa.
- [ ] Logs son fáciles de leer.
