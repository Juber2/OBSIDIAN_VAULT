---
tags:
  - Docker
  - DevOps
  - Docker-CICD
Creado: 2026-07-28
Relacionado:
  - CI CD
  - DOCKER
---

# 05 - Integracion con GitHub Actions

## Que vas a aprender

En este capitulo vas a consolidar todo el capitulo en un pipeline completo y realista usando **GitHub Actions**: la accion oficial `docker/build-push-action`, matrices de build multi-arquitectura, y buenas practicas especificas de esta plataforma.

## Explicacion

GitHub Actions ofrece acciones oficiales mantenidas por Docker (`docker/setup-buildx-action`, `docker/login-action`, `docker/build-push-action`, `docker/metadata-action`) que encapsulan gran parte de lo visto en este capitulo (cache remoto, multi-arch, tagging) en configuracion declarativa, en vez de scripts de shell manuales propensos a errores.

`docker/metadata-action` es especialmente util: genera automaticamente los tags apropiados (commit sha, rama, version semantica desde un tag de Git) siguiendo convenciones estandar, reduciendo la logica condicional manual que se vio en el tema anterior.

## Modelo mental

Pensa estas acciones oficiales como electrodomesticos especializados en vez de armar cada paso a mano con herramientas genericas: en vez de escribir vos mismo la logica de "que tag le pongo segun el evento", `docker/metadata-action` es la maquina que ya sabe hacer exactamente eso siguiendo convenciones probadas por miles de proyectos.

## Ejemplo base

```yaml
name: Build and Push
on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-buildx-action@v3

      - uses: docker/login-action@v3
        if: github.event_name != 'pull_request'
        with:
          username: ${{ secrets.REGISTRY_USER }}
          password: ${{ secrets.REGISTRY_TOKEN }}

      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: mi-registro/api
          tags: |
            type=sha,prefix=sha-
            type=semver,pattern={{version}}

      - uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

## Como se usa en un proyecto real

1. Se usan las acciones oficiales de Docker en vez de reimplementar logica de tagging y push con scripts manuales, reduciendo errores y facilitando mantenimiento.
2. Se condiciona el `push: true` explicitamente segun el evento (nunca publicar desde un pull request de un fork no confiable, por razones de seguridad).
3. Se configuran los secretos del registry (usuario, token) como GitHub Secrets, nunca en texto plano en el workflow.
4. Se combina `platforms:` para build multi-arquitectura junto con `cache-from`/`cache-to` de tipo `gha` (cache nativo de GitHub Actions) para builds rapidos y portables.
5. Se documenta el workflow completo como parte del repositorio, sirviendo de referencia para otros proyectos del equipo.

## Criterio junior

Un junior debe poder configurar un workflow basico de GitHub Actions que construya y publique una imagen usando las acciones oficiales de Docker.

## Criterio mid

Un mid debe usar `docker/metadata-action` para generar tags automaticamente segun convenciones, y condicionar el push segun el evento que disparo el pipeline.

## Criterio senior

Un senior debe diseñar el workflow estandar de CI/CD de Docker para toda la organizacion, incluyendo multi-arch, cache, seguridad de secretos, y firma de imagenes (ver capitulo de Registries), como plantilla reutilizable entre proyectos.

## Errores comunes

- Reimplementar manualmente logica de tagging con scripts de shell en vez de usar `docker/metadata-action`, aumentando el riesgo de errores y la complejidad de mantenimiento.
- Publicar imagenes desde pull requests de forks externos no confiables, arriesgando exponer secretos del registry a codigo no confiable.
- No usar el cache nativo de la plataforma (`type=gha`), perdiendo aceleracion de builds disponible sin costo adicional de configuracion.
- No condicionar correctamente los pasos de login y push segun el evento, intentando publicar en contextos donde no corresponde o donde no hay credenciales disponibles.

## Practica

- [ ] Configura un workflow de GitHub Actions completo usando las acciones oficiales de Docker.
- [ ] Usa `docker/metadata-action` para generar tags automaticamente segun el evento.
- [ ] Verifica que el push este condicionado correctamente para no publicar desde pull requests externos.
- [ ] Agrega build multi-arquitectura con cache nativo de GitHub Actions.
- [ ] Documenta este workflow como plantilla reutilizable para otros proyectos de tu equipo.

## Preguntas de repaso

- Que ventaja da `docker/metadata-action` sobre implementar manualmente la logica de tagging?
- Por que nunca se deberia publicar una imagen desde un pull request de un fork externo no confiable?
- Que combinacion de opciones usarias para build multi-arquitectura con cache eficiente en GitHub Actions?
- Como estructurarias un workflow para que sirva de plantilla reutilizable entre distintos proyectos del equipo?
