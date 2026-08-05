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

# 04 - Build y Push Automatizado con Tags

## Que vas a aprender

En este capitulo vas a integrar todo lo visto en un flujo completo de **build y push automatizado**: como disparar el pipeline segun el evento de Git (push, pull request, tag), y como generar tags de forma automatica y coherente (ver capitulo de Registries).

## Explicacion

Un pipeline de build y push bien diseñado distingue el comportamiento segun el **evento que lo disparo**: un push a una rama de feature podria solo construir y testear (sin publicar), un merge a la rama principal podria construir, testear y publicar con el tag del commit, y crear un tag de Git formal (`v1.4.0`) podria disparar ademas la publicacion de un tag de version semantica y potencialmente el deploy a produccion.

Esta distincion evita publicar imagenes de trabajo en progreso al mismo lugar que las imagenes que realmente se despliegan, manteniendo el registry organizado y la trazabilidad clara entre eventos de Git y artefactos publicados.

## Modelo mental

Pensa el pipeline como un sistema de clasificacion de correspondencia: no todo lo que se escribe (cada commit) se envia por correo (se publica); solo lo que pasa por ciertos buzones especificos (merge a main, tag de release) realmente sale a distribucion, y cada tipo de buzon determina el tipo de sello (tag) que lleva el envio.

## Ejemplo base

```yaml
# GitHub Actions: comportamiento distinto segun el evento
on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build (siempre, incluso en PRs)
        run: docker build -t test-build .

      - name: Push con tag de commit (solo en main)
        if: github.ref == 'refs/heads/main'
        run: |
          docker tag test-build mi-registro/api:sha-${{ github.sha }}
          docker push mi-registro/api:sha-${{ github.sha }}

      - name: Push con version semantica (solo en tags v*)
        if: startsWith(github.ref, 'refs/tags/v')
        run: |
          docker tag test-build mi-registro/api:${{ github.ref_name }}
          docker push mi-registro/api:${{ github.ref_name }}
```

## Como se usa en un proyecto real

1. Se construye y testea la imagen en cada pull request, sin publicarla, como validacion temprana antes de mergear.
2. Se construye, testea y publica con tag de commit en cada merge a la rama principal, siguiendo la estrategia de tagging trazable.
3. Se publica ademas con tag de version semantica solo cuando se crea un tag de Git formal, marcando releases oficiales.
4. Se conecta el evento de publicacion de version formal con el pipeline de deploy a produccion (ver capitulo de DevOps de NestJS para el otro lado de este flujo), cerrando el ciclo completo de CI/CD.
5. Se documenta claramente en el README del proyecto que evento de Git dispara cada comportamiento del pipeline.

## Criterio junior

Un junior debe poder configurar un pipeline basico que construya la imagen en cada push y la publique solo desde la rama principal.

## Criterio mid

Un mid debe diferenciar el comportamiento del pipeline segun el tipo de evento de Git (PR, merge, tag de release), aplicando la estrategia de tagging correspondiente a cada caso.

## Criterio senior

Un senior debe diseñar el flujo completo de CI/CD de la organizacion, conectando eventos de Git con publicacion de imagenes y despliegues, asegurando trazabilidad completa desde un commit hasta lo que corre en produccion.

## Errores comunes

- Publicar imagenes al registry en cada push de cualquier rama, ensuciando el registry con builds de trabajo en progreso.
- No distinguir el comportamiento del pipeline segun el evento, tratando pull requests igual que merges a la rama principal.
- No conectar los tags de version semantica con el proceso de deploy, perdiendo la oportunidad de automatizar el flujo completo release-a-produccion.
- No documentar que evento dispara que comportamiento, generando confusion sobre por que una imagen se publico o no.

## Practica

- [ ] Configura un pipeline que construya y testee la imagen en cada pull request sin publicarla.
- [ ] Configura la publicacion con tag de commit al mergear a la rama principal.
- [ ] Configura la publicacion con tag de version semantica al crear un tag de Git formal.
- [ ] Documenta en el README que evento de Git dispara cada comportamiento del pipeline.
- [ ] Diseña como conectarias la publicacion de una version formal con el disparo de un deploy a produccion.

## Preguntas de repaso

- Por que conviene construir y testear en cada pull request sin publicar la imagen al registry?
- Que diferencia de comportamiento tendria el pipeline entre un merge a main y la creacion de un tag de release?
- Como asegurarias trazabilidad completa desde un commit especifico hasta la imagen que corre en produccion?
- Que evento de Git usarias como disparador de un deploy automatico a produccion?
