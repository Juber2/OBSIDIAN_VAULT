---
tags:
  - Docker
  - DevOps
  - Docker-Optimizacion
Creado: 2026-07-28
Relacionado:
  - Optimizacion de Imagenes
  - DOCKER
---

# 03 - BuildKit y Cache Avanzado

## Que vas a aprender

En este capitulo vas a profundizar en **BuildKit**, el motor de build moderno de Docker (activado por defecto en versiones recientes), y sus features avanzadas de cache: **cache mounts** y ejecucion **paralela** de etapas independientes.

## Explicacion

BuildKit reemplaza al motor de build legacy con mejoras sustanciales: builds paralelos de etapas que no dependen entre si en un multi-stage build, mejor cache (incluyendo cache mounts, que persisten entre builds distintos sin quedar como capa de la imagen final), y soporte para BuildKit secrets (ya visto en el capitulo de Seguridad).

Un **cache mount** (`--mount=type=cache`) es especialmente util para directorios de cache de gestores de paquetes (`~/.npm`, `~/.cache/pip`): a diferencia del cache de capas normal (que se invalida completamente si cambia el contexto), un cache mount persiste su contenido entre builds **independientemente** de si la instruccion se re-ejecuta, acelerando instalaciones incluso cuando el lockfile cambio.

## Modelo mental

Pensa el cache de capas tradicional como una fotocopia completa que se reusa solo si es identica a una anterior. Pensa un cache mount como una carpeta de trabajo permanente en el escritorio que persiste entre sesiones de trabajo distintas, independiente de si el resto del trabajo cambio: siempre esta ahi para reusar lo que ya se descargo antes, aunque el resultado final sea distinto.

## Ejemplo base

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci
COPY . .
RUN npm run build
```

```bash
# BuildKit habilitado explicitamente (default en versiones modernas)
DOCKER_BUILDKIT=1 docker build -t mi-imagen .

# Construir multiples plataformas y aprovechar cache remoto compartido en CI
docker buildx build --cache-from type=registry,ref=mi-registro/cache \
  --cache-to type=registry,ref=mi-registro/cache -t mi-imagen .
```

## Como se usa en un proyecto real

1. Se usan cache mounts para directorios de cache de gestores de paquetes, acelerando instalaciones incluso cuando el lockfile cambia parcialmente.
2. Se aprovecha la ejecucion paralela automatica de BuildKit para etapas de multi-stage build que no dependen entre si (por ejemplo, build del backend y build del frontend en paralelo).
3. Se configura cache remoto compartido (`--cache-from`/`--cache-to` con un registry) en pipelines de CI, para que builds en agentes distintos (o efimeros) puedan reusar cache entre ejecuciones.
4. Se mide el tiempo de build antes y despues de adoptar cache mounts, para verificar la mejora real en el contexto especifico del proyecto.
5. Se documenta la sintaxis `# syntax=docker/dockerfile:1` al inicio del Dockerfile para asegurar que se usan las features mas recientes de BuildKit disponibles.

## Criterio junior

Un junior debe poder usar un cache mount basico para acelerar la instalacion de dependencias en un Dockerfile.

## Criterio mid

Un mid debe configurar cache remoto compartido en un pipeline de CI para reusar cache entre builds en agentes efimeros distintos.

## Criterio senior

Un senior debe optimizar la estrategia de build de todo el equipo aprovechando ejecucion paralela y cache avanzado de BuildKit, midiendo el impacto real en tiempos de CI/CD como metrica de productividad del equipo.

## Errores comunes

- No usar cache mounts para dependencias pesadas, perdiendo aceleracion posible incluso cuando el cache de capas tradicional se invalida.
- No configurar cache compartido en CI con agentes efimeros, perdiendo todo el beneficio del cache en cada build desde cero.
- Asumir que BuildKit esta siempre activo sin verificarlo en entornos de CI mas antiguos que puedan usar el motor legacy.
- No aprovechar la paralelizacion automatica de etapas independientes en multi-stage builds complejos.

## Practica

- [ ] Agrega un cache mount para el gestor de paquetes de tu proyecto y mide la mejora en tiempo de build.
- [ ] Verifica que BuildKit esta activo en tu entorno de build local y de CI.
- [ ] Configura cache remoto compartido con un registry para un pipeline de CI de ejemplo.
- [ ] Identifica si tu multi-stage build tiene etapas independientes que BuildKit podria paralelizar automaticamente.
- [ ] Compara tiempos de build con y sin las optimizaciones de BuildKit aplicadas.

## Preguntas de repaso

- Que diferencia hay entre el cache de capas tradicional y un cache mount de BuildKit?
- Por que un cache mount puede acelerar una instalacion incluso cuando el lockfile cambio?
- Que ventaja da configurar cache remoto compartido en un pipeline de CI con agentes efimeros?
- Que tipo de etapas de un multi-stage build puede paralelizar automaticamente BuildKit?
