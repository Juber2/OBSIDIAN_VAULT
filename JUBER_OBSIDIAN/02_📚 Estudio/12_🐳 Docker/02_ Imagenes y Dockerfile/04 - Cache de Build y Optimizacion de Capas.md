---
tags:
  - Docker
  - DevOps
  - Docker-Imagenes
Creado: 2026-07-28
Relacionado:
  - Imagenes y Dockerfile
  - DOCKER
---

# 04 - Cache de Build y Optimizacion de Capas

## Que vas a aprender

En este capitulo vas a aprender a **ordenar un Dockerfile** para maximizar el aprovechamiento del cache de build de Docker, reduciendo drasticamente el tiempo de build en iteraciones sucesivas.

## Explicacion

Docker cachea el resultado de cada instruccion como una capa; en un build posterior, si una instruccion y su contexto (los archivos que usa) no cambiaron respecto al build anterior, Docker reutiliza la capa cacheada en vez de re-ejecutar la instruccion. El problema es que este cache es **secuencial e invalidable en cascada**: si una instruccion cambia, todas las instrucciones siguientes en el Dockerfile pierden su cache, sin importar si su contenido especifico cambio o no.

La implicancia practica mas importante es el orden: las instrucciones que cambian con **menos frecuencia** (instalar dependencias del sistema, instalar dependencias de la aplicacion desde el lockfile) deben ir **antes** que las que cambian con mucha frecuencia (copiar el codigo fuente de la aplicacion). Por eso el patron estandar es copiar primero `package.json`/lockfile, instalar dependencias, y **recien despues** copiar el resto del codigo fuente.

## Modelo mental

Pensa el cache de Docker como un dominó: si tirás la primera ficha (una instruccion cambia), todas las fichas que siguen despues tambien caen (pierden cache), sin importar si esas fichas en particular hubieran quedado igual. Por eso conviene poner las fichas que casi nunca se mueven al principio, y las que cambian todo el tiempo al final.

## Ejemplo base

```dockerfile
# Mal: cualquier cambio de codigo invalida el cache de npm ci
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN npm ci
CMD ["node", "dist/main.js"]

# Bien: npm ci se cachea mientras el lockfile no cambie
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["node", "dist/main.js"]
```

## Como se usa en un proyecto real

1. Se copian primero los archivos de definicion de dependencias (`package.json`, lockfile) y se instalan dependencias, antes de copiar el resto del codigo fuente.
2. Se agrupan instrucciones `RUN` relacionadas con `&&` cuando tiene sentido, para reducir el numero de capas sin perder legibilidad.
3. Se usa BuildKit (activado por defecto en versiones modernas de Docker) que ademas soporta cache mounts para acelerar instalaciones de dependencias entre builds incluso en CI.
4. Se mide el tiempo de build antes y despues de reordenar instrucciones, para verificar la mejora real en vez de asumirla.
5. Se documenta en el Dockerfile, con un comentario breve, por que el orden de ciertas instrucciones es deliberado (cuando no es obvio).

## Criterio junior

Un junior debe poder identificar en un Dockerfile existente si el orden de instrucciones aprovecha bien el cache, y reordenar un caso simple.

## Criterio mid

Un mid debe medir el impacto real del cache en tiempos de build, y usar cache mounts de BuildKit para acelerar instalaciones de dependencias.

## Criterio senior

Un senior debe estandarizar patrones de Dockerfile que maximicen cache para todo el equipo, y balancear la legibilidad del Dockerfile contra el numero de capas y la granularidad del cache.

## Errores comunes

- Copiar todo el codigo fuente antes de instalar dependencias, invalidando el cache de instalacion en cada cambio de codigo.
- No medir el impacto real del reordenamiento, asumiendo mejoras que en la practica no son significativas para ese proyecto.
- Combinar demasiadas instrucciones en un unico `RUN` gigante, dificultando el debugging de que paso especifico fallo.
- No aprovechar cache mounts de BuildKit para dependencias pesadas que se reinstalan en cada build de CI sin necesidad.

## Practica

- [ ] Reordena un Dockerfile existente para copiar dependencias antes que el codigo fuente.
- [ ] Mide el tiempo de build antes y despues del reordenamiento, con un cambio menor de codigo entre ambos.
- [ ] Investiga como usar cache mounts de BuildKit para acelerar `npm ci` o equivalente.
- [ ] Agrupa instrucciones `RUN` relacionadas de forma legible.
- [ ] Anota una regla de tu equipo sobre el orden de instrucciones en cualquier Dockerfile nuevo.

## Preguntas de repaso

- Por que el cache de Docker se invalida "en cascada" a partir de la primera instruccion que cambia?
- Por que conviene copiar el lockfile de dependencias antes que el resto del codigo fuente?
- Que ventaja adicional dan los cache mounts de BuildKit sobre el cache de capas tradicional?
- Como medirias objetivamente si un reordenamiento de Dockerfile realmente mejoro los tiempos de build?
