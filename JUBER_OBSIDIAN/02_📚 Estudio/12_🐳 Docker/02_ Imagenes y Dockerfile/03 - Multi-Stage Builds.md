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

# 03 - Multi-Stage Builds

## Que vas a aprender

En este capitulo vas a dominar **multi-stage builds**: como separar la etapa de compilacion de la etapa final de runtime dentro de un unico Dockerfile, para producir imagenes de produccion mas chicas y seguras.

## Explicacion

Un multi-stage build usa multiples instrucciones `FROM` en el mismo Dockerfile, cada una iniciando una **etapa** nombrable (`AS builder`). El contenido de una etapa anterior puede copiarse selectivamente a una etapa posterior con `COPY --from=builder`, dejando atras todo lo que no se copio explicitamente: compiladores, dependencias de desarrollo, codigo fuente sin compilar.

El resultado es que la imagen final solo contiene lo estrictamente necesario para ejecutar la aplicacion (el build ya compilado, las dependencias de produccion), sin arrastrar herramientas de build que aumentan el tamaño y la superficie de ataque sin necesidad en el ambiente donde realmente corre la aplicacion.

## Modelo mental

Pensa un multi-stage build como cocinar en una cocina industrial (la etapa de build, con todos los utensilios y equipamiento pesado) pero servir el plato final en un plato simple sobre la mesa del comedor (la etapa final): el cliente nunca ve ni necesita ver toda la cocina que se uso para preparar el plato.

## Ejemplo base

```dockerfile
# Etapa 1: build
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Etapa 2: produccion (imagen final, mucho mas chica)
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/main.js"]
```

## Como se usa en un proyecto real

1. Se usa multi-stage siempre que el proceso de build requiera herramientas (compiladores, dependencias de desarrollo) que no deben existir en produccion.
2. Se nombran las etapas explicitamente (`AS builder`, `AS test`) para que `COPY --from` sea legible y explicito sobre su origen.
3. Se pueden agregar etapas intermedias adicionales, por ejemplo una etapa `test` que corre la suite de tests, sin que eso afecte el tamaño de la imagen final si no se copia nada de esa etapa.
4. Se copian solo los artefactos estrictamente necesarios entre etapas (`dist/`, no todo `/app`), minimizando lo que termina en la imagen final.
5. Se puede usar una etapa base compartida entre build y produccion para evitar declarar dos veces `FROM` con la misma version.

## Criterio junior

Un junior debe poder escribir un multi-stage build simple con una etapa de build y una etapa final que copia el resultado.

## Criterio mid

Un mid debe diseñar multi-stage builds con etapas intermedias (por ejemplo, testing) y copiar selectivamente solo los artefactos necesarios entre etapas.

## Criterio senior

Un senior debe estandarizar el patron de multi-stage build para todo el equipo, midiendo el impacto real en tamaño de imagen y tiempo de build, y balanceando la complejidad del Dockerfile contra el beneficio obtenido.

## Errores comunes

- Copiar toda la carpeta de la etapa de build (`COPY --from=builder /app .`) en vez de solo los artefactos necesarios, arrastrando basura innecesaria a la imagen final.
- No nombrar las etapas, generando un Dockerfile dificil de leer cuando hay mas de dos `FROM`.
- Olvidar que cada etapa es independiente: una variable de entorno o archivo de una etapa no existe automaticamente en otra si no se copia o redeclara explicitamente.
- No aprovechar multi-stage para separar tests del build final, perdiendo la oportunidad de que el Dockerfile documente tambien como se testea la aplicacion.

## Practica

- [ ] Convierte un Dockerfile de una sola etapa en un multi-stage build.
- [ ] Compara el tamaño de la imagen final antes y despues del cambio.
- [ ] Agrega una etapa intermedia de testing que no afecte el tamaño de la imagen final.
- [ ] Nombra explicitamente cada etapa y usa `COPY --from` con el nombre en vez de un indice numerico.
- [ ] Anota que artefactos minimos necesita copiar tu aplicacion desde la etapa de build.

## Preguntas de repaso

- Que ventaja practica da un multi-stage build sobre un Dockerfile de una sola etapa?
- Que pasa con el contenido de una etapa que nunca se copia a otra etapa posterior?
- Por que conviene nombrar explicitamente las etapas de un multi-stage build?
- Como usarias una etapa intermedia para correr tests sin que afecte el tamaño final de la imagen?
