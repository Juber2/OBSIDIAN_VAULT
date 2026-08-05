---
tags:
  - PostgreSQL
  - Cloud
  - Postgres-Cloud
Creado: 2026-07-29
Relacionado:
  - Postgres en la Nube y Contenedores
  - POSTGRESQL
---

# 01 - Postgres en Docker

## Que vas a aprender

En este capitulo vas a consolidar todo lo ya visto en el libro de Docker (capitulo de Bases de Datos para Desarrollo) especificamente aplicado a Postgres: volumenes persistentes, health checks reales, y como este conocimiento se conecta con la administracion ya aprendida en este libro.

## Explicacion

Correr Postgres en Docker para desarrollo (ya visto en detalle en el libro de Docker) requiere aplicar exactamente los mismos principios ya estudiados aca: el directorio de datos (`/var/lib/postgresql/data`) debe montarse en un **volumen nombrado** (nunca un bind mount, por los problemas de permisos ya explicados en el libro de Docker), un **health check real** deberia verificar con `pg_isready` que el motor esta aceptando conexiones (no solo que el proceso arranco), y las variables de entorno (`POSTGRES_PASSWORD`, `POSTGRES_DB`) configuran la inicializacion del container.

Para produccion, correr Postgres en un container propio (en vez de un servicio gestionado, ver siguiente tema) es una decision valida si el equipo tiene experiencia operando bases de datos con estado en containers, aplicando ademas todo lo aprendido en este capitulo: backups automatizados desde el container, replicacion entre containers Postgres, y monitoreo de recursos coordinado con los limites de container ya vistos en el libro de Docker.

## Modelo mental

Pensa correr Postgres en Docker como aplicar un traje hecho a medida (todo lo aprendido en este libro sobre administracion de Postgres) sobre un cuerpo distinto (la capa de containerizacion ya estudiada en el libro de Docker): las medidas y principios del traje no cambian, solo el contexto de despliegue donde se aplican.

## Ejemplo base

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: mi_app
    volumes:
      - datos_postgres:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  datos_postgres:
```

```bash
# Backup desde un container Postgres, aplicando el patron ya visto en el libro de Docker
docker exec postgres pg_dump -U postgres -Fc mi_app > backup.dump
```

## Como se usa en un proyecto real

1. Se usa el patron completo ya visto en el libro de Docker (volumen nombrado, health check con `pg_isready`, scripts de inicializacion) como base para cualquier Postgres en desarrollo local.
2. Se aplican los conceptos de este libro (backup con `pg_dump`, roles con menor privilegio, configuracion de `postgresql.conf`) exactamente igual dentro de un container Postgres que en una instalacion nativa.
3. Se evalua correr Postgres en container en produccion solo si el equipo tiene experiencia real operando bases de datos con estado en ese contexto, aplicando el marco de decision del capitulo Senior del libro de Docker.
4. Se coordina el dimensionamiento de recursos del container (`--memory`, ver capitulo de Costos del libro de Docker) con la configuracion interna de `shared_buffers` de Postgres (ver capitulo de Arquitectura de este libro).
5. Se conecta explicitamente este tema con ambos libros, siendo el punto de union directo entre "administrar Postgres" y "administrar containers".

## Criterio junior

Un junior debe poder levantar Postgres en Docker con persistencia correcta, aplicando el patron ya visto en el libro de Docker.

## Criterio mid

Un mid debe configurar backups, roles, y health checks reales para un Postgres en Docker, coordinando ambos libros de conocimiento.

## Criterio senior

Un senior debe decidir con criterio cuando Postgres en container es apropiado para produccion, coordinando dimensionamiento de recursos de container con configuracion interna del motor.

## Errores comunes

- No aplicar el patron de volumen nombrado ya visto en el libro de Docker, arriesgando perdida de datos o problemas de permisos con bind mounts.
- No coordinar los limites de memoria del container Docker con `shared_buffers` de Postgres, generando configuraciones internas incoherentes con el limite externo real disponible.
- Correr Postgres en container en produccion sin la experiencia operativa necesaria, subestimando la complejidad de administrar una base de datos con estado en ese contexto.
- No aplicar toda la administracion ya aprendida en este libro (backups, roles, VACUUM) solo porque el motor corre dentro de un container.

## Practica

- [ ] Levanta Postgres en Docker aplicando el patron completo de volumen nombrado y health check ya visto en el libro de Docker.
- [ ] Ejecuta un backup con `pg_dump` desde dentro del container.
- [ ] Configura un rol de aplicacion con permisos minimos dentro de ese Postgres en Docker.
- [ ] Coordina el limite de memoria del container con la configuracion de `shared_buffers` interna.
- [ ] Evalua si correrias Postgres en container o nativo para un proyecto de produccion real tuyo.

## Preguntas de repaso

- Que patron de volumen deberia usarse siempre para el directorio de datos de Postgres en Docker?
- Como verificarias con un health check real que Postgres esta listo para aceptar conexiones dentro de un container?
- Por que coordinar el limite de memoria del container con `shared_buffers` interno es importante?
- Que experiencia operativa necesitaria tu equipo antes de correr Postgres en container en produccion?
