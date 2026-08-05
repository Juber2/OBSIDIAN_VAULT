---
tags:
  - Docker
  - DevOps
  - Docker-Proyectos
Creado: 2026-07-28
Relacionado:
  - Proyectos
  - DOCKER
---

# 02 - Proyecto Stack Compose App DB Redis

## Que vas a construir

Un `docker-compose.yml` completo que levante la aplicacion dockerizada del proyecto anterior junto con una base de datos y Redis, aplicando los capitulos de Redes y Docker Compose.

## Alcance funcional

- Servicio de aplicacion, base de datos y Redis definidos en un unico `docker-compose.yml`.
- Segmentacion de red: base de datos y Redis en una red interna no publicada al host; solo la aplicacion publica un puerto.
- Health checks reales para base de datos y Redis, con la aplicacion esperando `condition: service_healthy` antes de arrancar.
- Variables de entorno parametrizadas con `.env` y `.env.example` versionado.
- Volumenes nombrados para persistencia de la base de datos.

## Checklist tecnica

- [ ] La base de datos y Redis no publican ningun puerto directamente al host (solo accesibles desde la red interna).
- [ ] La aplicacion espera correctamente a que la base de datos este `healthy`, no solo `started`, antes de intentar conectarse.
- [ ] `.env` real esta en `.gitignore`; `.env.example` esta versionado y documentado.
- [ ] Todo el stack levanta correctamente desde cero con un unico comando (`docker compose up`) en una maquina limpia.
- [ ] Los datos de la base de datos persisten correctamente a traves de `docker compose down` (sin `-v`) seguido de `up`.

## Criterio de exito

El proyecto se considera completo cuando otra persona puede clonar el repositorio, copiar `.env.example` a `.env`, y levantar el stack completo con un unico comando sin ningun paso manual adicional, obteniendo una aplicacion funcional conectada correctamente a su base de datos y cache.

## Errores comunes a evitar

- Publicar el puerto de la base de datos al host sin necesidad, rompiendo la segmentacion de seguridad.
- Usar `depends_on` simple sin `condition: service_healthy`, causando fallos intermitentes de conexion al arrancar.
- Commitear `.env` con valores reales en vez de solo `.env.example`.

## Extension opcional

Agregar un servicio de administracion (Adminer o similar) con un profile de Compose, disponible solo cuando se activa explicitamente para debugging.
