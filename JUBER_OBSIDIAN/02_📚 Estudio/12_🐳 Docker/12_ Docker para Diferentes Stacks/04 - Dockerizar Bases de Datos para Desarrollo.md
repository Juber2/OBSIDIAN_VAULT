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

# 04 - Dockerizar Bases de Datos para Desarrollo

## Que vas a aprender

En este capitulo vas a consolidar el patron practico para usar **bases de datos en containers durante desarrollo**: Postgres, SQL Server (relevante para proyectos .NET como StaffTrainingPlatform), Redis, y como inicializarlas con datos de ejemplo automaticamente.

## Explicacion

Para desarrollo local, correr Postgres, SQL Server o Redis en containers evita instalar y mantener esos motores directamente en la maquina de cada desarrollador, garantizando que todo el equipo use exactamente la misma version. Ademas de la persistencia con volumenes (ya visto en el capitulo de Volumenes), muchas imagenes oficiales de bases de datos soportan **scripts de inicializacion automatica**: archivos SQL o scripts colocados en una carpeta especifica que se ejecutan solo la primera vez que el volumen esta vacio, utiles para crear el schema base o cargar datos de ejemplo sin pasos manuales.

Para proyectos .NET con SQL Server especificamente, la imagen oficial de Microsoft requiere aceptar el EULA via variable de entorno y tiene requisitos minimos de memoria que conviene conocer antes de la primera ejecucion.

## Modelo mental

Pensa una base de datos en container para desarrollo como un electrodomestico de demostracion en una tienda: identico en comportamiento al que usarias en produccion, pero facil de resetear a su estado de fabrica (borrar el volumen) cuando queres empezar de cero sin arrastrar datos de pruebas anteriores.

## Ejemplo base

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: dev
    volumes:
      - datos-pg:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d   # se ejecuta solo la primera vez

  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      ACCEPT_EULA: "Y"
      MSSQL_SA_PASSWORD: "TuPassword123!"
    ports: ["1433:1433"]
    volumes:
      - datos-mssql:/var/opt/mssql

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

volumes:
  datos-pg:
  datos-mssql:
```

## Como se usa en un proyecto real

1. Se usan scripts de inicializacion (`docker-entrypoint-initdb.d` en Postgres/MySQL) para automatizar la creacion de schema base o datos de ejemplo en un entorno de desarrollo nuevo.
2. Se documenta el proceso de "resetear" la base de datos de desarrollo (`docker compose down -v` seguido de `up`) para cuando un desarrollador necesita empezar de cero.
3. Para SQL Server, se verifica que la maquina tenga suficiente memoria asignada a Docker (minimo recomendado por Microsoft) antes de reportar problemas de arranque como bugs.
4. Se mantiene la misma version de motor de base de datos entre el container de desarrollo y el ambiente de produccion real, minimizando sorpresas de comportamiento.
5. Se conecta esto con las migrations de la aplicacion (ver capitulo de Base de Datos en el libro de NestJS, o el equivalente en Entity Framework para .NET) que se ejecutan sobre esta base ya levantada.

## Criterio junior

Un junior debe poder levantar una base de datos en container para desarrollo local con persistencia via volumen.

## Criterio mid

Un mid debe usar scripts de inicializacion automatica para poblar una base de datos de desarrollo con schema o datos de ejemplo sin pasos manuales.

## Criterio senior

Un senior debe estandarizar el stack de bases de datos de desarrollo para todo el equipo, asegurando paridad de version con produccion y documentando el proceso completo de reset y troubleshooting.

## Errores comunes

- Instalar motores de base de datos directamente en la maquina de cada desarrollador en vez de usar containers, generando inconsistencias de version entre el equipo.
- No usar scripts de inicializacion, requiriendo pasos manuales repetitivos cada vez que alguien necesita un entorno de desarrollo nuevo.
- Asignar memoria insuficiente a Docker para correr SQL Server, causando fallos de arranque que se malinterpretan como bugs de configuracion.
- Usar una version de motor de base de datos en desarrollo distinta a la de produccion, generando sorpresas de compatibilidad al desplegar.

## Practica

- [ ] Levanta una base de datos (Postgres o SQL Server) en container con persistencia via volumen nombrado.
- [ ] Crea un script de inicializacion que se ejecute automaticamente la primera vez que el volumen esta vacio.
- [ ] Documenta el proceso de reset completo de la base de datos de desarrollo.
- [ ] Verifica que la version del motor en tu container de desarrollo coincide con la de produccion.
- [ ] Conecta esta base de datos con las migrations de tu aplicacion y verifica que corren correctamente.

## Preguntas de repaso

- Que ventaja da usar bases de datos en containers para desarrollo sobre instalarlas directamente en cada maquina?
- Para que sirven los scripts de inicializacion automatica de imagenes como Postgres?
- Que requisito especifico tiene la imagen oficial de SQL Server que hay que conocer antes del primer arranque?
- Por que mantener la misma version de motor entre desarrollo y produccion reduce sorpresas al desplegar?
