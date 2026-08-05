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

# 01 - Anatomia de docker-compose.yml

## Que vas a aprender

En este capitulo vas a aprender la estructura basica de un archivo **`docker-compose.yml`**: como se definen servicios, y como Compose orquesta multiples containers relacionados con un unico comando.

## Explicacion

Docker Compose resuelve el problema de tener que ejecutar manualmente varios comandos `docker run` largos y coordinados para levantar una aplicacion con multiples servicios (app, base de datos, cache). Un archivo `docker-compose.yml` declara, en YAML, cada **servicio** (que en la practica es una definicion de como correr un container: que imagen o Dockerfile usar, que puertos publicar, que variables de entorno definir, que volumenes montar), y Compose se encarga de crear la red compartida, construir las imagenes necesarias, y levantar todo en el orden correcto con un unico `docker compose up`.

Cada servicio definido se convierte en un container, y Compose automaticamente crea una red custom para el proyecto (ver capitulo de Redes) donde cada servicio puede alcanzar a los demas usando su nombre como hostname.

## Modelo mental

Pensa `docker-compose.yml` como el plano completo de un edificio de departamentos: en vez de construir departamento por departamento con instrucciones sueltas, tenes un unico documento que describe todo el edificio (todos los servicios), y un comando que lo construye entero de una vez, con las conexiones (red) ya resueltas entre departamentos.

## Ejemplo base

```yaml
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      DB_HOST: db
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: dev
    volumes:
      - datos-db:/var/lib/postgresql/data

volumes:
  datos-db:
```

```bash
docker compose up -d       # levanta todos los servicios en background
docker compose down        # detiene y elimina containers (los volumenes persisten)
docker compose logs -f api # sigue los logs de un servicio especifico
```

## Como se usa en un proyecto real

1. Se define un `docker-compose.yml` en la raiz del proyecto como forma estandar de levantar el entorno de desarrollo completo con un unico comando.
2. Se usa `build: .` para servicios propios (que se construyen desde un Dockerfile local) e `image: nombre:tag` para servicios de terceros (bases de datos, colas).
3. Se organizan las variables de entorno usando un archivo `.env` referenciado automaticamente por Compose (ver siguiente tema).
4. Se nombra el proyecto explicitamente (o se deja que Compose use el nombre del directorio) para que los recursos generados (red, volumenes) sean identificables.
5. Se versiona el `docker-compose.yml` en el repositorio, como parte de la documentacion ejecutable de como correr el proyecto.

## Criterio junior

Un junior debe poder escribir un `docker-compose.yml` basico con dos servicios relacionados y levantarlos con `docker compose up`.

## Criterio mid

Un mid debe estructurar un `docker-compose.yml` completo para un proyecto real, combinando servicios propios (build) y de terceros (image), con puertos, variables de entorno y volumenes correctamente configurados.

## Criterio senior

Un senior debe diseñar el `docker-compose.yml` de un proyecto grande pensando en mantenibilidad (uso de `.env`, overrides, ver siguientes temas) y en que sea la fuente de verdad ejecutable de como el sistema esta compuesto.

## Errores comunes

- Escribir comandos `docker run` largos y repetitivos manualmente en vez de declarar el mismo entorno en un `docker-compose.yml` reproducible.
- No versionar el `docker-compose.yml` en el repositorio, perdiendo la documentacion ejecutable del setup del proyecto.
- Mezclar servicios de desarrollo y produccion en un unico archivo sin ninguna separacion (ver tema de profiles y overrides).
- Olvidar que `docker compose down` sin `-v` preserva los volumenes, generando confusion sobre si los datos persistieron o no.

## Practica

- [ ] Escribe un `docker-compose.yml` con al menos dos servicios relacionados (app + base de datos).
- [ ] Levanta el entorno completo con `docker compose up -d` y verifica que los servicios se comunican entre si.
- [ ] Sigue los logs de un servicio especifico con `docker compose logs -f`.
- [ ] Verifica la diferencia entre `docker compose down` y `docker compose down -v`.
- [ ] Documenta en un README como levantar el proyecto usando este `docker-compose.yml`.

## Preguntas de repaso

- Que problema resuelve Docker Compose frente a ejecutar varios `docker run` manualmente?
- Que diferencia hay entre usar `build:` y `image:` para un servicio en Compose?
- Que crea Compose automaticamente que facilita la comunicacion entre servicios?
- Que pasa con los volumenes al correr `docker compose down` sin la flag `-v`?
