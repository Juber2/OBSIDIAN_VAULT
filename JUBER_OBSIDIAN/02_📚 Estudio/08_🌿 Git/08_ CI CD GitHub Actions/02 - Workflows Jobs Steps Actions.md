---
tags:
  - GitHubActions
Creado: 2026-05-22
Actualizado: 2026-08-09
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

## Dependencias entre jobs (`needs`)

Por defecto los jobs corren en paralelo. `needs` los ordena y hace que uno espere a otro:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "build"

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "test"

  deploy:
    needs: [build, test]
    runs-on: ubuntu-latest
    steps:
      - run: echo "deploy"
```

## Pasar datos entre jobs (`outputs`)

Cada job corre en un runner distinto (no comparten filesystem), así que para pasar un valor de un job a otro se usa `outputs`:

```yaml
jobs:
  set-version:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.getver.outputs.version }}
    steps:
      - id: getver
        run: echo "version=1.2.3" >> "$GITHUB_OUTPUT"

  deploy:
    needs: set-version
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying version ${{ needs.set-version.outputs.version }}"
```

## Condicionales (`if`)

```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - run: echo "solo corre en push a main"
```

Expresiones útiles:
- `success()` (default implícito), `failure()`, `always()`, `cancelled()`.
- `github.event_name`, `github.ref`, `github.actor`.

```yaml
  - name: Notificar si falla
    if: failure()
    run: echo "algo falló, notificar"
```

## Contextos más usados

| Contexto | Ejemplo |
|---|---|
| `github` | `github.sha`, `github.actor`, `github.event_name` |
| `env` | variables de entorno del workflow/job/step |
| `secrets` | `secrets.MY_TOKEN` |
| `vars` | `vars.API_URL` (variables no secretas) |
| `matrix` | valores de la estrategia matrix |
| `needs` | outputs de jobs previos |
| `steps` | outputs de steps previos dentro del mismo job |

## Timeouts y control de concurrencia

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 15   # evita jobs colgados consumiendo minutos
```

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true   # cancela el run anterior si llega un push nuevo a la misma rama
```

## Checklist
- [ ] Uso nombres claros.
- [ ] Defino eventos correctos.
- [ ] Uso `checkout`.
- [ ] Divido jobs si aporta claridad.
- [ ] Sé encadenar jobs con `needs` y pasar `outputs`.
- [ ] Uso `if` para condicionar ejecución.
- [ ] Configuro `timeout-minutes` y `concurrency` para evitar desperdicio de minutos.
