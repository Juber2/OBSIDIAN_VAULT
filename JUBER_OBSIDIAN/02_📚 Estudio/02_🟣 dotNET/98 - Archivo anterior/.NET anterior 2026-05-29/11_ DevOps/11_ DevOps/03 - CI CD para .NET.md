---
tags:
  - NET
  - CICD
Creado: 2026-05-21
Relacionado:
  - DevOps con .NET
---

# CI/CD para .NET

Un pipeline mínimo debe restaurar, compilar y testear.

```yaml
name: .NET CI
on: [push, pull_request]
jobs:
  build:
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

## Checklist

- [ ] Corre en cada PR.
- [ ] Tests fallidos bloquean merge.
- [ ] Publico artefacto versionado.
- [ ] No despliego desde laptop.
