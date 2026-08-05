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

# 03 - Variables de Entorno y .env en Compose

## Que vas a aprender

En este capitulo vas a aprender a manejar configuracion en Compose de forma limpia: el archivo `.env` que Compose lee automaticamente, interpolacion de variables dentro del YAML, y la diferencia entre variables usadas por Compose mismo y variables pasadas al container.

## Explicacion

Compose lee automaticamente un archivo `.env` en el mismo directorio que el `docker-compose.yml`, y permite **interpolar** esas variables dentro del propio YAML con la sintaxis `${VARIABLE}`, util para parametrizar tags de imagen, puertos, o cualquier valor que cambie entre entornos sin editar el archivo YAML en si.

Es importante distinguir dos usos distintos de variables: las que Compose usa para **interpolar el propio archivo YAML** (resueltas al parsear el archivo, antes de crear containers), y las que se pasan al container via `environment:` (que el proceso dentro del container lee en su propio `process.env` o equivalente). Confundir ambos casos genera errores donde una variable "no llega" al lugar esperado.

## Modelo mental

Pensa el `.env` leido por Compose como una lista de ingredientes que se usan para completar la receta (el YAML) antes de empezar a cocinar. Una vez que la receta esta completa con los ingredientes puestos en su lugar, `environment:` es lo que efectivamente le entregas al comensal (el container) para que lo use durante la comida.

## Ejemplo base

```bash
# .env (mismo directorio que docker-compose.yml)
IMAGE_TAG=1.4.0
API_PORT=3000
DB_PASSWORD=supersecreto
```

```yaml
services:
  api:
    image: mi-registro/api:${IMAGE_TAG}   # interpolacion resuelta por Compose al parsear
    ports:
      - "${API_PORT}:3000"
    environment:
      DB_PASSWORD: ${DB_PASSWORD}         # ahora SI llega al process.env del container
```

## Como se usa en un proyecto real

1. Se usa `.env` para parametrizar valores que cambian entre entornos (tag de imagen, puertos) sin tocar el `docker-compose.yml`.
2. Se mantiene un `.env.example` versionado en el repositorio (sin valores reales) documentando que variables existen, mientras que `.env` real nunca se commitea.
3. Se distingue claramente entre interpolar el YAML (`${VAR}` fuera de `environment:`) y pasar la variable al proceso del container (dentro de `environment:`).
4. Se usan archivos `.env` distintos por entorno (`.env.development`, `.env.production`) referenciados explicitamente con `--env-file` cuando se necesita mas de un set de configuracion.
5. Se evita interpolar secretos directamente en comandos visibles del shell; se prefiere que vivan solo en el archivo `.env` con permisos restringidos.

## Criterio junior

Un junior debe poder usar un archivo `.env` basico para parametrizar un valor en `docker-compose.yml` con interpolacion `${VAR}`.

## Criterio mid

Un mid debe distinguir claramente entre interpolacion de Compose y variables de entorno del container, y manejar multiples archivos `.env` por ambiente con `--env-file`.

## Criterio senior

Un senior debe definir la estrategia de gestion de configuracion para Compose en todo el equipo, asegurando que secretos reales nunca se commiteen y que la diferencia entre interpolacion y variables de container sea clara para todo el equipo.

## Errores comunes

- Commitear el archivo `.env` real con secretos al repositorio en vez de solo un `.env.example`.
- Confundir interpolacion de Compose con variables de entorno del container, sorprendiendose de que una variable "no llega" a la aplicacion.
- No mantener un `.env.example` actualizado, dificultando que un nuevo integrante sepa que variables necesita configurar.
- Usar el mismo `.env` para desarrollo y produccion sin separacion, arriesgando que configuracion de un entorno se filtre al otro.

## Practica

- [ ] Crea un `.env` y usalo para interpolar un valor en `docker-compose.yml`.
- [ ] Pasa una variable de ese `.env` al container via `environment:` y verifica que el proceso interno la recibe.
- [ ] Crea un `.env.example` documentando las variables sin exponer valores reales.
- [ ] Configura dos archivos `.env` distintos (desarrollo y produccion) y usa `--env-file` para elegir cual aplicar.
- [ ] Verifica que `.env` real esta en `.gitignore`.

## Preguntas de repaso

- Que diferencia hay entre una variable usada para interpolar el YAML y una pasada al container via `environment:`?
- Por que `.env` real nunca deberia commitearse, mientras que `.env.example` si?
- Como usarias `--env-file` para manejar configuracion distinta entre desarrollo y produccion?
- Que problema puede causar confundir interpolacion de Compose con variables de entorno del container?
