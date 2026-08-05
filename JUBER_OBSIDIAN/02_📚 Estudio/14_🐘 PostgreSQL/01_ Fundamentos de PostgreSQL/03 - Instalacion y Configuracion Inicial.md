---
tags:
  - PostgreSQL
  - Bases-de-Datos
  - Postgres-Fundamentos
Creado: 2026-07-29
Relacionado:
  - Fundamentos de PostgreSQL
  - POSTGRESQL
---

# 03 - Instalacion y Configuracion Inicial

## Que vas a aprender

En este capitulo vas a aprender las distintas formas de tener Postgres disponible para practicar y trabajar: instalacion nativa en Linux (conectando con el libro de Linux), via Docker (conectando con ese libro), y los archivos de configuracion clave (`postgresql.conf`, `pg_hba.conf`).

## Explicacion

Instalar Postgres nativamente en Linux se hace con el gestor de paquetes del sistema (`apt install postgresql` en Debian/Ubuntu, ya visto en el libro de Linux), quedando administrado como servicio systemd. La alternativa mas practica para desarrollo es correrlo en un container Docker (ya visto en detalle en el capitulo de Bases de Datos para Desarrollo del libro de Docker), evitando instalar el motor directamente en la maquina.

Dos archivos de configuracion son centrales: **`postgresql.conf`** controla el comportamiento del motor (memoria, conexiones, logging, todo lo visto en el capitulo anterior), y **`pg_hba.conf`** (host-based authentication) controla **quien puede conectarse desde donde y con que metodo de autenticacion**, siendo la primera linea de defensa de acceso a la base de datos, mas fundamental incluso que los permisos de usuario dentro de la base.

## Modelo mental

Pensa `postgresql.conf` como el manual de operacion interna del motor (como se comporta una vez que alguien ya entro), y `pg_hba.conf` como la lista de control de acceso en la puerta principal (quien tiene siquiera permiso de tocar el timbre, desde que direccion, y que credencial debe mostrar).

## Ejemplo base

```bash
# Instalacion nativa (Debian/Ubuntu, ver libro de Linux)
sudo apt install postgresql postgresql-contrib
sudo systemctl status postgresql

# Via Docker (ver libro de Docker, capitulo de Bases de Datos para Desarrollo)
docker run -d --name postgres -e POSTGRES_PASSWORD=dev -p 5432:5432 postgres:16
```

```
# pg_hba.conf: quien puede conectarse, desde donde, con que metodo
# TYPE  DATABASE  USER  ADDRESS         METHOD
host    all       all   127.0.0.1/32    scram-sha-256
host    all       all   10.0.0.0/24     scram-sha-256
```

## Como se usa en un proyecto real

1. Se usa Docker para desarrollo local (mas simple, mas rapido de resetear) y se evalua instalacion nativa o servicio gestionado para produccion (ver capitulo de Postgres en la Nube).
2. Se restringe `pg_hba.conf` a las direcciones y metodos de autenticacion estrictamente necesarios, nunca dejando `trust` (sin contraseña) en un entorno mas alla de desarrollo local aislado.
3. Se usa `scram-sha-256` como metodo de autenticacion por contraseña, el mas seguro disponible actualmente, en vez de metodos legacy mas debiles.
4. Se documenta la ubicacion de estos archivos de configuracion segun la distribucion (`/etc/postgresql/VERSION/main/` en Debian/Ubuntu), dado que varia entre sistemas.
5. Se reinicia o recarga (`pg_ctl reload`, o `systemctl reload postgresql`) despues de cambios de configuracion, aplicando el mismo principio de "verificar antes de aplicar" visto en el libro de Linux.

## Criterio junior

Un junior debe poder instalar Postgres nativamente o via Docker, y conectarse exitosamente con `psql`.

## Criterio mid

Un mid debe configurar `pg_hba.conf` restringiendo acceso apropiadamente, y ubicar los archivos de configuracion en su distribucion especifica.

## Criterio senior

Un senior debe estandarizar la configuracion inicial de Postgres para todo el equipo (metodo de autenticacion, restricciones de red), documentando el proceso como parte de la infraestructura como codigo (ver libro de Linux).

## Errores comunes

- Dejar `pg_hba.conf` con metodo `trust` (sin contraseña) en un entorno accesible mas alla de desarrollo local completamente aislado.
- No reiniciar o recargar el servicio despues de modificar archivos de configuracion, sorprendiendose de que los cambios no tienen efecto.
- Instalar Postgres directamente en la maquina de desarrollo cuando Docker daria un entorno mas simple de resetear entre pruebas.
- No documentar la ubicacion de los archivos de configuracion, perdiendo tiempo buscandolos en cada distribucion distinta.

## Practica

- [ ] Instala Postgres via Docker siguiendo el patron ya visto en el libro correspondiente.
- [ ] Conectate con `psql` y verifica la version instalada.
- [ ] Si tenes una instalacion nativa, ubica y revisa `postgresql.conf` y `pg_hba.conf`.
- [ ] Ajusta `pg_hba.conf` para restringir acceso a una direccion especifica con `scram-sha-256`.
- [ ] Aplica el cambio con un reload y verifica que sigue funcionando correctamente.

## Preguntas de repaso

- Que diferencia de proposito hay entre `postgresql.conf` y `pg_hba.conf`?
- Por que el metodo de autenticacion `trust` es peligroso fuera de desarrollo local aislado?
- Que ventaja da usar Docker para Postgres en desarrollo sobre una instalacion nativa?
- Que pasos son necesarios despues de modificar `pg_hba.conf` para que el cambio tenga efecto?
