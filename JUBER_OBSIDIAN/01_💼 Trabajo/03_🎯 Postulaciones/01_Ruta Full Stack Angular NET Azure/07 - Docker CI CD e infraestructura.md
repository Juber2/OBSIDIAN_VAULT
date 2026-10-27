---
tags:
  - Postulaciones
  - Docker
  - CICD
  - DevOps
Creado: 2026-08-30
Relacionado:
  - DOCKER
  - GIT
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Docker, CI/CD e infraestructura

## Resultado esperado

Un reclutador técnico debe poder clonar el repositorio, ejecutar un comando documentado y levantar frontend, API y dependencias.

## Docker imprescindible

- Imagen vs container y capas.
- Dockerfile multi-stage para .NET y Angular.
- `.dockerignore` y caché de build.
- Usuario no-root y superficie mínima.
- Variables, secrets y configuración por ambiente.
- Redes, volúmenes y DNS interno.
- Health checks y graceful shutdown.
- Docker Compose para desarrollo e integración.

Repasar: [[02 - Dockerizar una App .NET]], [[03 - Dockerizar un Frontend Angular]], [[03 - Multi-Stage Builds]], [[05 - Compose para un Stack Full-Stack]], [[04 - Health Checks]].

## Pipeline mínimo

```text
pull request
  ├─ restore/install
  ├─ lint + format
  ├─ unit tests
  ├─ integration tests
  ├─ build Angular + .NET
  ├─ build y scan de imágenes
  └─ publicar artefactos

main/tag
  ├─ push a registry
  ├─ deploy a ambiente
  ├─ smoke test
  └─ rollback si falla
```

Elegir GitHub Actions o Azure DevOps según las vacantes. Dominar uno y comprender el otro es mejor que tener dos pipelines superficiales.

Repasar: [[GitHub Actions CI CD]], [[CI CD build test publish]], [[05 - Integracion con GitHub Actions]], [[05 - Deploy con GitHub Actions]].

## Infraestructura como código

Aprender después del primer deploy manual:

- parámetros por ambiente;
- recursos, outputs y dependencias;
- secretos fuera del repositorio;
- preview/plan antes de aplicar;
- estado y drift si se usa Terraform.

Para un perfil Azure, Bicep ofrece entrada directa. Terraform suma portabilidad y aparece en muchas vacantes. Elegir uno para el proyecto y saber explicar el otro.

## Evidencia de calidad

- [ ] README con arquitectura y comandos.
- [ ] Build reproducible.
- [ ] Imágenes con tags trazables, no solo `latest`.
- [ ] Pipeline bloquea merge si fallan tests.
- [ ] Secrets no aparecen en Git ni en la imagen.
- [ ] Health checks distinguen proceso vivo y dependencias disponibles.
- [ ] Existe una estrategia de rollback.

## Errores que debes reconocer

- Ejecutar como root sin necesidad.
- Copiar todo el repositorio antes de restaurar dependencias, destruyendo la caché.
- Incluir `node_modules`, secretos o binarios locales.
- Usar Compose como orquestador de producción complejo.
- Desplegar aunque las pruebas fallen.
- Hacer migrations destructivas automáticamente sin estrategia.
