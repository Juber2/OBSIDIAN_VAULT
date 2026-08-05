---
tags:
  - NestJS
  - TypeScript
  - Nest-DevOps
Creado: 2026-07-27
Relacionado:
  - DevOps Deploy
  - NESTJS
---

# 02 - Docker para NestJS

## Que vas a aprender

En este capitulo vas a aprender a contenerizar una aplicacion NestJS con Docker de forma eficiente: **multi-stage builds** para imagenes chicas y seguras, y `docker-compose` para levantar el stack completo (app + base de datos + Redis) en desarrollo.

## Explicacion

Un `Dockerfile` naive (una sola etapa, copiando todo el codigo fuente y `node_modules` de desarrollo) genera imagenes innecesariamente grandes y con herramientas de build que no deberian existir en produccion (compiladores TypeScript, dependencias de desarrollo). Un **multi-stage build** separa la etapa de compilacion (instala todas las dependencias, corre `nest build`) de la etapa final (solo copia el `dist/` compilado y las dependencias de produccion), resultando en una imagen mucho mas chica y con menor superficie de ataque.

`docker-compose` es la herramienta tipica para desarrollo local: levanta la app Nest junto con sus dependencias (Postgres, Redis) en containers separados mediante un solo comando, replicando de forma simplificada la topologia de produccion.

## Modelo mental

Pensa el multi-stage build como cocinar en una cocina grande con todos los utensilios (etapa de build) pero servir el plato final en un plato limpio y minimalista (etapa de produccion), sin llevar toda la cocina a la mesa del cliente.

## Ejemplo base

```dockerfile
# Etapa de build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Etapa final
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
USER node
CMD ["node", "dist/main.js"]
```

## Como se usa en un proyecto real

1. Se usa multi-stage build siempre, separando dependencias de desarrollo de las de produccion en la imagen final.
2. Se corre el proceso como usuario no-root (`USER node`) dentro del container, reduciendo el impacto de una eventual vulnerabilidad.
3. `docker-compose.yml` levanta app, base de datos y Redis en desarrollo, con volumenes para persistir datos entre reinicios.
4. Se usa `.dockerignore` para no copiar `node_modules`, `.git` ni archivos innecesarios dentro del contexto de build.
5. Se etiquetan las imagenes con el hash del commit o version semantica, nunca solo `latest`, para poder rastrear que codigo esta corriendo en cada ambiente.

## Criterio junior

Un junior debe poder escribir un `Dockerfile` basico de una etapa y levantar la aplicacion con `docker-compose up`.

## Criterio mid

Un mid debe implementar un multi-stage build que reduzca el tamaño final de la imagen, y configurar `docker-compose` con las dependencias necesarias para desarrollo local.

## Criterio senior

Un senior debe optimizar las imagenes para produccion (tamaño, seguridad, usuario no-root), definir la estrategia de tagging y versionado de imagenes, y coordinar la configuracion de Docker con el pipeline de CI/CD.

## Errores comunes

- Copiar `node_modules` de desarrollo (con devDependencies) a la imagen final de produccion, aumentando tamaño y superficie de ataque innecesariamente.
- Correr el proceso como root dentro del container sin necesidad.
- No usar `.dockerignore`, incluyendo archivos innecesarios o sensibles en el contexto de build.
- Etiquetar imagenes solo como `latest`, perdiendo trazabilidad de que version esta corriendo en cada ambiente.

## Practica

- [ ] Escribe un `Dockerfile` multi-stage para un proyecto Nest de practica.
- [ ] Configura `docker-compose.yml` con app, base de datos y Redis.
- [ ] Compara el tamaño de la imagen antes y despues de aplicar multi-stage build.
- [ ] Configura el container para correr como usuario no-root.
- [ ] Anota tu convencion de tagging de imagenes para produccion.

## Preguntas de repaso

- Que ventaja da un multi-stage build sobre un Dockerfile de una sola etapa?
- Por que correr el proceso como usuario no-root dentro del container es una buena practica?
- Para que sirve `docker-compose` en el flujo de desarrollo local?
- Que riesgo tiene etiquetar imagenes solo como `latest`?
