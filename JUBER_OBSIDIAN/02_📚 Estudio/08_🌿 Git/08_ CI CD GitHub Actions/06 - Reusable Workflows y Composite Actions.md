---
tags:
  - GitHubActions
  - Avanzado
Creado: 2026-08-09
Relacionado:
  - GitHub Actions CI CD
---

# Reusable Workflows y Composite Actions

Cuando varios repos (o varios workflows del mismo repo) repiten los mismos steps, conviene extraer esa lógica en vez de copiar/pegar YAML. Hay dos mecanismos distintos para esto, y confundirlos es un error común.

## Reusable Workflows (`workflow_call`)

Un workflow completo que **otro workflow invoca como si fuera un job**. Vive igual en `.github/workflows/`, pero se dispara con `workflow_call` en vez de (o además de) `push`/`pull_request`.

`.github/workflows/build-reusable.yml`:
```yaml
name: Build reusable

on:
  workflow_call:
    inputs:
      dotnet-version:
        required: true
        type: string
    secrets:
      NUGET_TOKEN:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ inputs.dotnet-version }}
      - run: dotnet build
```

Otro workflow lo consume:
```yaml
jobs:
  build:
    uses: ./.github/workflows/build-reusable.yml
    with:
      dotnet-version: "8.0.x"
    secrets:
      NUGET_TOKEN: ${{ secrets.NUGET_TOKEN }}
```

También se puede referenciar un reusable workflow de **otro repositorio**:
```yaml
uses: mi-org/workflows-compartidos/.github/workflows/build-reusable.yml@main
```

## Composite Actions

Un conjunto de **steps** empaquetados como una action propia, con su carpeta y un `action.yml`. A diferencia del reusable workflow, esto no es un job completo — se usa **dentro** de un step existente, mezclado con otros steps del mismo job.

`.github/actions/setup-proyecto/action.yml`:
```yaml
name: "Setup proyecto"
description: "Instala dependencias y cachea"
inputs:
  node-version:
    required: true
    default: "20"
runs:
  using: "composite"
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
    - run: npm ci
      shell: bash
```

Uso en un workflow:
```yaml
steps:
  - uses: actions/checkout@v4
  - uses: ./.github/actions/setup-proyecto
    with:
      node-version: "20"
  - run: npm test
```

## Cuándo usar cada uno

| Necesito... | Uso |
|---|---|
| Reutilizar un pipeline completo (varios jobs) entre repos | Reusable workflow |
| Estandarizar 2-4 steps que se repiten dentro de jobs distintos | Composite action |
| Compartir lógica compleja en otro lenguaje (Node, Docker) | Custom action (JavaScript o Docker action) |

## Custom Actions en JavaScript/Docker (mención)

Para lógica más compleja que un composite de steps de shell, existen:
- **JavaScript actions**: corren directo en el runner (Node.js), sin overhead de contenedor.
- **Docker actions**: empaquetan cualquier lenguaje/herramienta en una imagen, más lentas de arrancar pero máxima flexibilidad.

Se documentan con `action.yml` (`runs.using: node20` o `runs.using: docker`) y se publican en el Marketplace si son públicas.

## Beneficio real: mantenimiento centralizado

Si mañana cambia la versión de Node o se agrega un paso de seguridad obligatorio, con reusable workflows/composite actions se edita **un solo lugar** y todos los repos que lo consumen lo heredan en su próximo run — evita el problema de "until copié este YAML a 10 repos y ahora hay que actualizar los 10".

## Checklist
- [ ] Distingo reusable workflow (job completo) de composite action (steps sueltos).
- [ ] Extraje a reusable/composite lo que se repite en 2+ workflows.
- [ ] Versiono los reusables (`@main` vs `@v1` con tags) según qué tan estable necesito que sea.
- [ ] Sé pasar `inputs` y `secrets` a un reusable workflow.
