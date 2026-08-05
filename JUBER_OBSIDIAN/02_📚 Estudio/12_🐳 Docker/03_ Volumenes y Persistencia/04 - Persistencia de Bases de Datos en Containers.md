---
tags:
  - Docker
  - DevOps
  - Docker-Volumenes
Creado: 2026-07-28
Relacionado:
  - Volumenes y Persistencia
  - DOCKER
---

# 04 - Persistencia de Bases de Datos en Containers

## Que vas a aprender

En este capitulo vas a aprender las consideraciones especificas para correr **bases de datos con estado** dentro de containers: performance de I/O, el driver de almacenamiento correcto, y por que correr una base de datos en container en produccion es una decision que merece analisis particular.

## Explicacion

Una base de datos es una carga de trabajo especialmente sensible a I/O de disco y consistencia de datos, lo cual la hace un caso particular dentro del mundo de containers. Un volume gestionado por Docker (con el driver `local` por defecto) da buena performance en Linux nativo, pero en Docker Desktop (Windows/Mac) la capa de virtualizacion puede introducir latencia adicional, relevante para bases de datos con alta carga de escritura.

Ademas, correr una base de datos productiva dentro de un container introduce preguntas especificas: como se manejan los backups (ver tema anterior), como se hace un upgrade de version mayor del motor sin downtime, y como se garantiza que un reinicio del container no corrompa datos a medio escribir. Por estas razones, muchos equipos optan por bases de datos gestionadas (RDS, Cloud SQL) en produccion, reservando containers de base de datos principalmente para desarrollo local y entornos de test.

## Modelo mental

Pensa una base de datos en un container como guardar objetos muy fragiles en una caja que, en teoria, podes mover facilmente de lugar: es comodo y practico, pero cada movimiento (reinicio, upgrade, migracion) tiene mas riesgo que con un archivo comun, y por eso conviene manejarlo con mas cuidado del habitual.

## Ejemplo base

```yaml
# docker-compose.yml: base de datos con volumen nombrado para desarrollo
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: dev
    volumes:
      - datos-db:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s

volumes:
  datos-db:
```

## Como se usa en un proyecto real

1. Se usa un volume nombrado (no un bind mount) para el directorio de datos de la base de datos, evitando problemas de permisos y performance de bind mounts en Windows/Mac.
2. Se define un `healthcheck` para que otros servicios (la aplicacion) esperen a que la base de datos este realmente lista, no solo a que el container haya arrancado.
3. Se evalua conscientemente si la base de datos de produccion corre en container propio o en un servicio gestionado, segun el equipo y los requisitos de disponibilidad.
4. Se planifican los upgrades de version mayor del motor de base de datos como una operacion cuidadosa, con backup previo y ventana de mantenimiento, nunca como un simple cambio de tag de imagen sin mas.
5. Se usan bases de datos en containers libremente en desarrollo y CI, donde la perdida de datos no tiene consecuencias reales.

## Criterio junior

Un junior debe poder levantar una base de datos en container con un volume persistente para desarrollo local, sin perder datos entre reinicios.

## Criterio mid

Un mid debe configurar health checks apropiados para bases de datos en container, y entender las diferencias de performance de I/O entre Linux nativo y Docker Desktop.

## Criterio senior

Un senior debe decidir con criterio cuando una base de datos de produccion corre en container propio versus un servicio gestionado, y diseñar el proceso seguro de upgrade de version mayor de un motor de base de datos containerizado.

## Errores comunes

- Usar un bind mount en vez de un volume nombrado para el directorio de datos de una base de datos, arriesgando problemas de permisos entre el usuario del container y el del host.
- No definir un health check, dejando que la aplicacion intente conectarse antes de que la base de datos este realmente lista para recibir conexiones.
- Cambiar el tag de imagen de una base de datos a una version mayor sin plan de migracion ni backup previo.
- Asumir que containerizar una base de datos de produccion es identico a containerizar una aplicacion sin estado, sin considerar los riesgos particulares.

## Practica

- [ ] Levanta una base de datos en container con un volume nombrado persistente.
- [ ] Configura un health check para esa base de datos.
- [ ] Reinicia el container de la base de datos y verifica que los datos persisten.
- [ ] Investiga el proceso recomendado de upgrade de version mayor para el motor de base de datos que usas.
- [ ] Anota tu criterio para decidir entre base de datos en container propio vs servicio gestionado en produccion.

## Preguntas de repaso

- Por que conviene un volume nombrado sobre un bind mount para el directorio de datos de una base de datos?
- Que problema resuelve un health check en el contexto de una base de datos en container?
- Que riesgos particulares tiene un upgrade de version mayor de un motor de base de datos containerizado?
- Que factores considerarias para decidir entre base de datos en container propio vs un servicio gestionado en produccion?
