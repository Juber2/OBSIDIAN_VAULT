---
tags:
  - GitHubActions
  - CICD
Creado: 2026-05-22
Actualizado: 2026-08-09
Relacionado:
  - GitHub Actions CI CD
---

# Fundamentos GitHub Actions

GitHub Actions automatiza tareas como build, tests, lint, análisis y despliegues, disparadas por eventos del repositorio.

## Conceptos base

| Concepto | Significado |
|---|---|
| Workflow | automatización definida en YAML, vive en `.github/workflows/*.yml` |
| Event | disparador: push, PR, schedule, workflow_dispatch, release, etc. |
| Job | grupo de steps que corre en un runner |
| Step | comando o action individual dentro de un job |
| Runner | máquina (hosted o self-hosted) que ejecuta los jobs |
| Action | unidad reutilizable de código (propia, de terceros o del marketplace) |

## Eventos comunes (`on:`)

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: "0 3 * * *"        # cron UTC, corre todos los días 3am
  workflow_dispatch:            # trigger manual desde la UI/CLI
    inputs:
      environment:
        description: "Ambiente destino"
        required: true
        default: "staging"
  release:
    types: [published]
```

- `workflow_dispatch` es clave para deploys manuales controlados.
- `schedule` usa sintaxis cron, siempre en UTC (ojo con la diferencia horaria).
- Se puede filtrar por `paths:` para que el workflow solo corra si cambian ciertos archivos (útil en monorepos).

```yaml
on:
  push:
    paths:
      - "src/**"
      - "!**.md"
```

## Runners

- **Hosted por GitHub**: `ubuntu-latest`, `windows-latest`, `macos-latest`. Gratis con límites de minutos según el plan (público ilimitado, privado con cuota mensual).
- **Self-hosted**: máquina propia registrada como runner. Útil para acceso a red interna, hardware específico o evitar límites de minutos.

## Caching y artifacts (para no repetir instalaciones)

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ hashFiles('package-lock.json') }}
```

- **Cache**: acelera instalaciones repetidas (node_modules, paquetes NuGet, etc.) — se invalida cambiando la `key`.
- **Artifacts** (`actions/upload-artifact` / `download-artifact`): pasan archivos entre jobs o los dejan disponibles para descarga (ej. reportes de test, binarios compilados).

## Matrix builds

Corre el mismo job contra varias combinaciones (versiones de lenguaje, SO, etc.) en paralelo:

```yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
    os: [ubuntu-latest, windows-latest]
runs-on: ${{ matrix.os }}
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node-version }}
```

## Marketplace

Miles de actions reutilizables (`actions/checkout`, `actions/setup-node`, `docker/build-push-action`, etc.). Antes de usar una de terceros: revisar estrellas/mantenimiento, y **pinnear por SHA** en vez de por tag mutable si es crítico para seguridad.

```yaml
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

## Checklist
- [ ] Sé dónde van los workflows.
- [ ] Entiendo eventos (push, PR, schedule, workflow_dispatch).
- [ ] Entiendo jobs y steps.
- [ ] Sé usar cache y artifacts.
- [ ] Sé armar un matrix build.
- [ ] Sé leer logs y re-ejecutar jobs fallidos.
