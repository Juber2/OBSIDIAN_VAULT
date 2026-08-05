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

# 05 - Compose para Stacks de Desarrollo

## Que vas a aprender

En este capitulo vas a consolidar todo lo anterior armando un **stack completo de desarrollo** con Compose: aplicacion propia, base de datos, cache, y las practicas que hacen que ese entorno sea rapido de levantar y facil de mantener para todo el equipo.

## Explicacion

El objetivo final de dominar Compose es poder darle a cualquier integrante del equipo (o a vos mismo en una maquina nueva) la capacidad de levantar el entorno de desarrollo completo con un unico comando, sin instalar manualmente Postgres, Redis, ni configurar nada a mano. Esto elimina buena parte del tiempo de onboarding y de los problemas de "en mi maquina no funciona" relacionados con versiones distintas de herramientas instaladas localmente.

Un stack de desarrollo bien diseñado combina: bind mounts para hot-reload del codigo propio (ver capitulo de Volumenes), named volumes para persistencia de datos entre reinicios, variables de entorno parametrizadas con `.env`, health checks para coordinar el arranque, y profiles para herramientas opcionales de soporte.

## Modelo mental

Pensa un buen stack de desarrollo con Compose como un kit de armado completo: todas las piezas (servicios) ya vienen definidas y conectadas correctamente, y armar el modelo completo (levantar el entorno) es un unico paso, en vez de tener que conseguir e instalar cada pieza por separado uno mismo.

## Ejemplo base

```yaml
services:
  api:
    build:
      context: .
      target: development
    volumes:
      - ./src:/app/src        # hot-reload del codigo
      - /app/node_modules     # evita que el bind mount pise node_modules del container
    ports: ["3000:3000"]
    environment:
      DB_HOST: db
    depends_on:
      db: { condition: service_healthy }

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: dev
    volumes: ["datos-db:/var/lib/postgresql/data"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]

  redis:
    image: redis:7-alpine

  adminer:
    image: adminer
    profiles: ["debug"]
    ports: ["8081:8080"]

volumes:
  datos-db:
```

## Como se usa en un proyecto real

1. Se documenta en el README un unico comando (`docker compose up`) como forma estandar de levantar el entorno completo de desarrollo.
2. Se usa el truco de montar `/app/node_modules` como volumen anonimo separado para que el bind mount del codigo fuente no sobreescriba las dependencias instaladas dentro del container con las del host (potencialmente incompatibles).
3. Se agregan herramientas de soporte (adminer, mailhog) como servicios con profile, disponibles pero no siempre activos.
4. Se mide el tiempo que tarda un desarrollador nuevo en tener el entorno completo funcionando, como metrica de calidad del setup.
5. Se mantiene el stack de desarrollo lo mas parecido posible al de produccion (mismas versiones de imagenes base) para minimizar sorpresas al desplegar.

## Criterio junior

Un junior debe poder levantar un stack de desarrollo completo ya definido por el equipo con un unico comando, sin necesitar instalar nada localmente mas alla de Docker.

## Criterio mid

Un mid debe diseñar un stack de desarrollo completo para un proyecto nuevo, combinando hot-reload, persistencia de datos, y herramientas de soporte opcionales.

## Criterio senior

Un senior debe optimizar la experiencia de desarrollo de todo el equipo con Compose, midiendo tiempo de onboarding, manteniendo paridad razonable entre desarrollo y produccion, y evolucionando el stack a medida que el proyecto crece en complejidad.

## Errores comunes

- No usar el patron de volumen anonimo para `node_modules`, causando que el bind mount del host sobreescriba dependencias instaladas dentro del container.
- Levantar herramientas de soporte pesadas siempre, sin usar profiles, ralentizando el arranque del entorno para el uso diario.
- Mantener el stack de desarrollo muy distinto al de produccion (versiones de imagenes distintas), generando sorpresas al desplegar.
- No documentar el comando estandar de arranque, dejando que cada desarrollador arme su propio flujo distinto.

## Practica

- [ ] Arma un stack de desarrollo completo con hot-reload, base de datos persistente y una herramienta de soporte con profile.
- [ ] Verifica el patron de volumen anonimo para evitar que el bind mount pise las dependencias instaladas en el container.
- [ ] Mide cuanto tiempo tarda levantar el entorno completo desde cero.
- [ ] Documenta el comando estandar de arranque en el README del proyecto.
- [ ] Compara las versiones de imagenes base entre tu stack de desarrollo y el de produccion.

## Preguntas de repaso

- Que problema resuelve montar `node_modules` como volumen anonimo separado del bind mount del codigo fuente?
- Por que usar profiles para herramientas de soporte mejora la experiencia diaria del equipo?
- Que beneficio de onboarding da un stack de desarrollo completo levantable con un unico comando?
- Por que mantener paridad de versiones entre desarrollo y produccion reduce sorpresas al desplegar?
