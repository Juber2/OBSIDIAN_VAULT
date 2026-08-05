---
tags:
  - GitHubActions
Creado: 2026-05-22
Relacionado:
  - GitHub Actions CI CD
---

# Workflows, Jobs, Steps y Actions

Un workflow vive en `.github/workflows`.

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Build"
```

## Checklist

- [ ] Uso nombres claros.
- [ ] Defino eventos correctos.
- [ ] Uso `checkout`.
- [ ] Divido jobs si aporta claridad.
