---
tags:
  - Docker
  - DevOps
  - Docker-Compose
Creado: 2026-07-28
Relacionado:
  - Docker Compose
  - DOCKER
---

# 04 - Profiles y Overrides

## Que vas a aprender

En este capitulo vas a aprender dos mecanismos de Compose para manejar variaciones entre entornos sin duplicar todo el archivo: **profiles** (activar servicios opcionales bajo demanda) y **archivos de override** (`docker-compose.override.yml`, o multiples archivos combinados con `-f`).

## Explicacion

Los **profiles** permiten marcar ciertos servicios como opcionales, que solo se levantan si se activa explicitamente ese profile (`docker compose --profile debug up`). Es util para servicios que solo se necesitan en ciertos escenarios: una herramienta de administracion de base de datos, un servicio de mocking, un profiler, que no deberian levantarse en cada `docker compose up` normal.

Los **archivos de override** permiten separar la configuracion base (compartida entre entornos) de las diferencias especificas: Compose automaticamente fusiona `docker-compose.yml` con `docker-compose.override.yml` si este ultimo existe en el mismo directorio, o se puede ser explicito combinando varios archivos con `-f base.yml -f produccion.yml`. Esto evita mantener archivos completos duplicados con pequeñas diferencias entre desarrollo y produccion.

## Modelo mental

Pensa el archivo base de Compose como los cimientos y la estructura general de un edificio, y cada archivo de override como una capa de personalizacion que se agrega encima segun donde se va a usar ese edificio (decoracion de oficina vs decoracion de vivienda), sin tener que reconstruir toda la estructura de nuevo para cada caso.

## Ejemplo base

```yaml
# docker-compose.yml (base, compartido)
services:
  api:
    build: .
    environment:
      NODE_ENV: production

  adminer:
    image: adminer
    profiles: ["debug"]   # NO se levanta con un `up` normal
```

```yaml
# docker-compose.override.yml (aplicado automaticamente en desarrollo local)
services:
  api:
    environment:
      NODE_ENV: development
    volumes:
      - ./src:/app/src   # bind mount solo en desarrollo
```

```bash
docker compose up                       # usa base + override automaticamente
docker compose --profile debug up       # ademas levanta adminer
docker compose -f docker-compose.yml -f docker-compose.prod.yml up  # combinacion explicita para produccion
```

## Como se usa en un proyecto real

1. Se usa `docker-compose.override.yml` (nombre reconocido automaticamente) para diferencias de desarrollo local, manteniendo el archivo base limpio y generico.
2. Se usan profiles para herramientas de soporte (administradores de DB, mailhog, profilers) que no deben levantarse siempre.
3. Se combinan archivos explicitamente con `-f` para produccion, evitando que el override de desarrollo se aplique accidentalmente ahi.
4. Se documenta en el README que combinacion de archivos y profiles corresponde a cada entorno.
5. Se evita duplicar servicios completos entre archivos; solo se sobreescriben los campos que realmente difieren.

## Criterio junior

Un junior debe poder usar un `docker-compose.override.yml` basico para agregar una diferencia de desarrollo local sobre el archivo base.

## Criterio mid

Un mid debe usar profiles para servicios opcionales y combinar explicitamente archivos con `-f` para distintos entornos.

## Criterio senior

Un senior debe diseñar la estrategia completa de composicion de archivos Compose para todos los entornos del equipo (desarrollo, CI, staging, produccion), documentando claramente que combinacion usar en cada caso.

## Errores comunes

- Duplicar el `docker-compose.yml` completo para cada entorno en vez de usar overrides, generando archivos que se desincronizan con el tiempo.
- No usar profiles para herramientas opcionales, haciendo que cada `docker compose up` levante servicios que rara vez se necesitan.
- Aplicar accidentalmente el override de desarrollo en produccion por no ser explicito con los archivos combinados (`-f`).
- No documentar claramente en el README que comando corresponde a cada entorno, generando confusion en el equipo.

## Practica

- [ ] Crea un `docker-compose.override.yml` con una diferencia de desarrollo sobre el archivo base.
- [ ] Agrega un servicio opcional con un profile y verifica que no se levanta en un `up` normal.
- [ ] Levanta ese servicio explicitamente activando el profile.
- [ ] Crea un archivo separado para produccion y combinalo explicitamente con `-f`.
- [ ] Documenta en un README los comandos correctos para cada entorno de tu proyecto.

## Preguntas de repaso

- Que archivo combina Compose automaticamente con el `docker-compose.yml` base, sin necesitar `-f` explicito?
- Para que sirve un profile y como se activa?
- Por que conviene usar overrides en vez de duplicar archivos completos por entorno?
- Que riesgo hay en no ser explicito sobre que archivos se combinan en produccion?
