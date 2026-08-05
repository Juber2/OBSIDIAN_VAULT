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

# 05 - Compose para un Stack Full-Stack

## Que vas a aprender

En este capitulo vas a integrar todos los temas anteriores del capitulo en un unico **stack full-stack completo**: frontend Angular, backend (NestJS o .NET), base de datos, y cache, todo orquestado con un solo `docker-compose.yml`.

## Explicacion

Este es el ejercicio de sintesis del capitulo: combinar la dockerizacion de cada pieza individual (backend, frontend, base de datos) vista en los temas anteriores dentro de una topologia de red coherente (capitulo de Redes), con dependencias correctamente coordinadas via health checks (capitulo de Redes, tema de comunicacion entre containers), y variables de entorno consistentes entre servicios (capitulo de Compose).

La arquitectura tipica coloca un reverse proxy (o el propio Nginx del frontend) como unico punto de entrada publico, con el backend y la base de datos en una red interna no expuesta directamente, siguiendo el principio de menor privilegio de red visto en el capitulo de Redes.

## Modelo mental

Pensa este stack completo como el edificio de departamentos completo del capitulo de Redes, ahora con inquilinos reales y especificos: el frontend en la puerta de calle (unico acceso publico), el backend en un piso intermedio (accesible desde la puerta pero no directamente desde la calle), y la base de datos en el sotano (solo accesible para el backend, nunca directamente desde afuera).

## Ejemplo base

```yaml
services:
  frontend:
    build: ./frontend
    networks: [publica]
    ports: ["80:80"]

  api:
    build: ./api
    networks: [publica, interna]
    environment:
      DB_HOST: db
      REDIS_HOST: redis
    depends_on:
      db: { condition: service_healthy }
      redis: { condition: service_started }

  db:
    image: postgres:16
    networks: [interna]
    environment:
      POSTGRES_PASSWORD: dev
    volumes: ["datos-db:/var/lib/postgresql/data"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]

  redis:
    image: redis:7-alpine
    networks: [interna]

networks:
  publica:
  interna:

volumes:
  datos-db:
```

## Como se usa en un proyecto real

1. Se define un unico `docker-compose.yml` en la raiz del proyecto (o del monorepo) que levanta el stack completo con un comando.
2. Se segmenta la red entre publica (frontend, y el backend si necesita ser alcanzado directo) e interna (base de datos, cache), aplicando lo visto en el capitulo de Redes.
3. Se coordinan las dependencias con health checks reales, no solo orden de arranque simple, evitando fallos de conexion intermitentes al levantar el stack.
4. Se documenta en el README un unico comando (`docker compose up`) como forma estandar de levantar el proyecto completo, cerrando el objetivo practico de todo el libro.
5. Se mantiene este stack de desarrollo con paridad razonable de versiones respecto a produccion, minimizando sorpresas al desplegar.

## Criterio junior

Un junior debe poder levantar un stack full-stack completo (frontend + backend + base de datos) con un unico `docker-compose.yml` siguiendo un ejemplo dado.

## Criterio mid

Un mid debe diseñar la topologia de red y las dependencias coordinadas de un stack full-stack nuevo desde cero, aplicando los capitulos de Redes y Compose.

## Criterio senior

Un senior debe estandarizar este patron de stack completo para todos los proyectos full-stack del equipo, sirviendo como plantilla de referencia que combina las buenas practicas de todo el libro en un unico artefacto ejecutable.

## Errores comunes

- Exponer la base de datos directamente a la red publica sin necesidad, rompiendo la segmentacion de seguridad vista en el capitulo de Redes.
- No coordinar correctamente las dependencias con health checks, generando fallos intermitentes de conexion al levantar el stack por primera vez.
- No documentar el comando estandar de arranque, dejando que cada desarrollador arme su propio flujo distinto para levantar el mismo proyecto.
- Dejar que el stack de desarrollo se desincronice de las versiones reales de produccion con el tiempo, sin revisarlo periodicamente.

## Practica

- [ ] Integra un frontend Angular, un backend (NestJS o .NET), y una base de datos en un unico `docker-compose.yml`.
- [ ] Segmenta la red entre publica e interna, verificando que la base de datos no es alcanzable directamente desde afuera.
- [ ] Configura health checks reales para coordinar correctamente el arranque de todos los servicios.
- [ ] Documenta el comando estandar de arranque en el README del proyecto.
- [ ] Verifica que todo el stack levanta correctamente desde cero con un unico comando en una maquina nueva.

## Preguntas de repaso

- Como aplicarias la segmentacion de red (publica/interna) a un stack full-stack completo?
- Que problema evitan los health checks reales al coordinar el arranque de un stack con multiples dependencias?
- Que beneficio de onboarding da documentar un unico comando estandar de arranque para todo el proyecto?
- Como mantendrias paridad de versiones entre este stack de desarrollo y el ambiente de produccion real?
