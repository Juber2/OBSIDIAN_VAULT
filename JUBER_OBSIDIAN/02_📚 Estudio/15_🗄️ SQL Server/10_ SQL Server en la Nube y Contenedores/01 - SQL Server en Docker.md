---
tags:
  - SQLServer
  - Nube
  - SQLServer-Nube
Creado: 2026-07-29
Relacionado:
  - SQL Server en la Nube y Contenedores
  - SQL SERVER
---

# 01 - SQL Server en Docker

## Que vas a aprender

En este capitulo vas a aprender a correr SQL Server en Docker (imagen oficial `mcr.microsoft.com/mssql/server`, disponible para Linux desde SQL Server 2017), aplicando directamente todo lo ya aprendido sobre contenedores en el libro de Docker.

## Explicacion

Microsoft distribuye una imagen oficial de SQL Server para Linux que corre en Docker de forma identica en espiritu a la imagen oficial de `postgres` ya usada extensamente en el libro de Docker: variables de entorno para configuracion inicial (`ACCEPT_EULA`, `MSSQL_SA_PASSWORD`), un volumen para persistir los archivos `.mdf`/`.ldf` fuera del ciclo de vida del contenedor, y exposicion del puerto `1433` (el puerto TCP estandar de SQL Server, analogo al `5432` de Postgres).

Correr SQL Server en Docker es la forma estandar de trabajar en desarrollo local hoy (reemplazando instalaciones nativas de SQL Server Express en la maquina de cada desarrollador), y tambien una opcion viable en produccion dentro de Kubernetes para cargas que no requieren Always On AG con almacenamiento compartido complejo.

## Modelo mental

Pensa la imagen oficial de SQL Server en Docker exactamente igual que la imagen oficial de Postgres ya dominada en el libro de Docker: mismo patron de variables de entorno para configuracion inicial, mismo patron de volumen para persistencia, mismo puerto expuesto conceptualmente (solo el numero cambia). Si ya sabes correr Postgres en Docker, correr SQL Server es aplicar el mismo modelo mental con nombres de variables distintos.

## Ejemplo base

```bash
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=ContraseñaSegura123!" \
    -p 1433:1433 \
    -v sqlserver_data:/var/opt/mssql \
    --name sqlserver-dev \
    -d mcr.microsoft.com/mssql/server:2022-latest
```

```yaml
# docker-compose.yml — mismo patron que Postgres ya visto en el libro de Docker
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      ACCEPT_EULA: "Y"
      MSSQL_SA_PASSWORD: "ContraseñaSegura123!"
    ports:
      - "1433:1433"
    volumes:
      - sqlserver_data:/var/opt/mssql

volumes:
  sqlserver_data:
```

## Como se usa en un proyecto real

1. Se usa SQL Server en Docker como base de datos de desarrollo local para StaffTrainingPlatform (.NET + EF Core), evitando instalar SQL Server nativamente en cada maquina de desarrollo.
2. Se define en `docker-compose.yml` junto con el resto de servicios del proyecto (API .NET, frontend), aplicando el mismo patron multi-servicio ya visto en el libro de Docker.
3. Se usa un volumen nombrado para persistir los datos entre reinicios del contenedor, evitando perder la base de datos de desarrollo local.
4. Se conecta esto directamente con el capitulo "Bases de Datos para Desarrollo" del libro de Docker, donde ya se establecio este mismo patron con Postgres.
5. Se documenta la contraseña de `sa` de desarrollo de forma segura (variables de entorno, nunca hardcodeada en el compose file versionado).

## Criterio junior

Un junior debe poder correr SQL Server en Docker con `docker run` y conectarse desde una herramienta cliente.

## Criterio mid

Un mid debe definir SQL Server en un `docker-compose.yml` con volumen persistente como parte de un stack de desarrollo completo.

## Criterio senior

Un senior debe evaluar cuando SQL Server en contenedores es apropiado para produccion (cargas sin necesidad de Always On AG complejo) versus cuando se justifica una instancia nativa o un servicio administrado.

## Errores comunes

- No usar un volumen persistente, perdiendo la base de datos de desarrollo cada vez que se recrea el contenedor.
- Hardcodear la contraseña de `sa` directamente en un `docker-compose.yml` versionado en el repositorio.
- No asignar suficiente memoria al contenedor (SQL Server tiene requisitos minimos de RAM mas altos que Postgres tipicamente).
- Asumir que SQL Server en Docker es identico en produccion a una instalacion nativa sin evaluar las diferencias reales.

## Practica

- [ ] Corre SQL Server en Docker con `docker run` y conectate con una herramienta cliente (Azure Data Studio o similar).
- [ ] Define SQL Server en un `docker-compose.yml` con volumen persistente.
- [ ] Verifica que los datos persisten despues de recrear el contenedor.
- [ ] Compara este proceso con correr Postgres en Docker, ya dominado en el libro de Docker.
- [ ] Conecta un proyecto .NET local a este SQL Server en Docker.

## Preguntas de repaso

- Que variables de entorno minimas necesita la imagen oficial de SQL Server para arrancar?
- Por que es importante usar un volumen persistente para SQL Server en Docker?
- Como se compara este proceso con correr Postgres en Docker?
- En que escenario de produccion seria apropiado correr SQL Server en un contenedor?
