---
tags:
  - Java
  - CI-CD
Creado: 2026-05-21
Relacionado:
  - DevOps con Java
---

# CI/CD para Java

CI/CD automatiza validaciones y despliegues. En Java, lo minimo es compilar, ejecutar tests, analizar calidad y generar artefacto.

## Pipeline minimo

```yaml
name: Java CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
      - run: ./mvnw clean verify
```

## Etapas recomendadas

| Etapa | Proposito |
|---|---|
| Compile | Detectar errores de build |
| Unit tests | Validar logica aislada |
| Integration tests | Validar DB, HTTP, colas |
| Static analysis | Calidad y seguridad |
| Package | Crear jar o imagen |
| Deploy | Publicar en ambiente |

## Checklist

- [ ] El pipeline corre en cada PR.
- [ ] Nadie despliega manualmente desde su laptop.
- [ ] Fallar tests bloquea merge.
- [ ] El artefacto desplegado es el mismo que fue probado.
