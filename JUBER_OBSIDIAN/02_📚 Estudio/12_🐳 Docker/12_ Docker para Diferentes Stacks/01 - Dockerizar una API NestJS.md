---
tags:
  - Docker
  - DevOps
  - Docker-Stacks
Creado: 2026-07-28
Relacionado:
  - Docker para Diferentes Stacks
  - DOCKER
---

# 01 - Dockerizar una API NestJS

## Que vas a aprender

En este capitulo vas a aplicar todo lo aprendido para dockerizar correctamente una **API NestJS**, conectando este libro con el [[NESTJS|libro de NestJS]]: multi-stage build especifico para Nest, manejo de variables de entorno con `ConfigModule`, y health check real.

## Explicacion

Una API NestJS se beneficia directamente de multi-stage build (capitulo de Imagenes): una etapa compila TypeScript a JavaScript (`nest build`), y la etapa final solo corre el resultado compilado con las dependencias de produccion, sin el compilador de TypeScript ni las devDependencies. La validacion de variables de entorno vista en el capitulo de DevOps del libro de NestJS (`ConfigModule` con schema estricto) se combina naturalmente con lo visto aca sobre `.env` y Compose: la aplicacion falla rapido y claro si el container arranca sin la configuracion necesaria.

El health check real (usando `@nestjs/terminus`, visto en el capitulo de Observabilidad de NestJS) se conecta directamente con la instruccion `HEALTHCHECK` del Dockerfile o con `condition: service_healthy` en Compose, cerrando el circulo entre ambos libros.

## Modelo mental

Pensa esta nota como el punto de union entre los dos libros: todo lo que aprendiste sobre construir una API NestJS solida se combina con todo lo que aprendiste sobre empaquetarla correctamente en un container, para producir un artefacto de despliegue completo y production-ready.

## Ejemplo base

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
RUN addgroup -g 1001 app && adduser -D -u 1001 -G app app
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder --chown=app:app /app/dist ./dist
USER app
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD node -e "require('http').get('http://localhost:3000/health', r => process.exit(r.statusCode === 200 ? 0 : 1))"
CMD ["node", "dist/main.js"]
```

## Como se usa en un proyecto real

1. Se aplica multi-stage build siempre, dado que TypeScript necesita compilarse pero el resultado final no necesita el compilador.
2. Se conecta el health check del Dockerfile con el endpoint `/health` real construido con `@nestjs/terminus` en la aplicacion.
3. Se configura `ConfigModule` para fallar rapido si el container arranca sin variables de entorno criticas, en vez de arrancar "a medias".
4. Se usa un usuario no-root (ver capitulo de Seguridad) para el proceso de Node dentro del container.
5. Se conecta este Dockerfile con el pipeline de CI/CD (capitulo de CI/CD de este libro) para automatizar build, test y push.

## Criterio junior

Un junior debe poder dockerizar una API NestJS basica con multi-stage build siguiendo este patron.

## Criterio mid

Un mid debe conectar el health check del container con un endpoint de salud real de NestJS, y validar configuracion de entorno correctamente.

## Criterio senior

Un senior debe estandarizar este Dockerfile como plantilla reutilizable para todos los servicios NestJS del equipo, integrado con el pipeline de CI/CD completo.

## Errores comunes

- No usar multi-stage build, arrastrando el compilador de TypeScript y devDependencies a la imagen de produccion.
- No conectar el `HEALTHCHECK` del Dockerfile con un endpoint de salud real de la aplicacion.
- Dejar el proceso corriendo como root dentro del container.
- No validar variables de entorno criticas, dejando que la aplicacion arranque de forma inconsistente sin configuracion completa.

## Practica

- [ ] Dockeriza una API NestJS real con multi-stage build siguiendo este patron.
- [ ] Conecta el `HEALTHCHECK` con un endpoint `/health` real construido con `@nestjs/terminus`.
- [ ] Verifica que el container falla rapido si falta una variable de entorno critica.
- [ ] Confirma que el proceso corre como usuario no-root.
- [ ] Mide el tamaño final de la imagen y aplica optimizaciones del capitulo correspondiente si es necesario.

## Preguntas de repaso

- Por que una API NestJS se beneficia especialmente de un multi-stage build?
- Como conectarias el `HEALTHCHECK` del Dockerfile con el sistema de health checks de NestJS?
- Que pasa si el container arranca sin las variables de entorno criticas validadas por `ConfigModule`?
- Que elementos de seguridad (visto en el capitulo de Seguridad de este libro) aplicarias a este Dockerfile?
