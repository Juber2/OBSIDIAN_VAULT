---
tags:
  - SQLServer
  - Bases-de-Datos
  - SQLServer-Fundamentos
Creado: 2026-07-29
Relacionado:
  - Fundamentos de SQL Server
  - SQL SERVER
---

# 05 - Bases de Datos y Archivos mdf ldf

## Que vas a aprender

En este capitulo vas a entender la estructura fisica de una base de datos SQL Server: los archivos **`.mdf`** (datos primarios), **`.ndf`** (datos secundarios opcionales), y **`.ldf`** (transaction log), y como se relacionan con los conceptos ya vistos de Postgres.

## Explicacion

Cada base de datos SQL Server esta compuesta fisicamente por al menos dos archivos: el **`.mdf`** (Master Data File) contiene los datos primarios y objetos del sistema, y el **`.ldf`** (Log Data File) contiene el Transaction Log (ya visto en el capitulo de Arquitectura, el equivalente al WAL de Postgres). Opcionalmente, archivos **`.ndf`** (secondary Data Files) permiten distribuir los datos en multiples archivos fisicos, util para distribuir I/O entre discos distintos en bases de datos grandes.

Esto contrasta con Postgres, donde cada tabla y sus indices son archivos individuales dentro de un directorio de datos compartido por toda la instancia (ver capitulo de FHS del libro de Linux, aplicado a la estructura de Postgres): SQL Server organiza la base de datos completa en un numero mas pequeño de archivos consolidados, cada uno pudiendo crecer segun configuracion de auto-crecimiento (`AUTOGROWTH`).

## Modelo mental

Pensa el `.mdf` como el archivador principal de una oficina donde viven todos los documentos activos, y el `.ldf` como el libro de registro cronologico de cada cambio hecho a esos documentos, usado para poder reconstruir el estado exacto si algo se pierde. Los archivos `.ndf` son archivadores adicionales opcionales para cuando el archivador principal se queda sin espacio o se quiere distribuir la carga entre varios muebles fisicos distintos.

## Ejemplo base

```sql
CREATE DATABASE MiApp
ON PRIMARY (
    NAME = 'MiApp_Data',
    FILENAME = 'C:\Data\MiApp.mdf',
    SIZE = 100MB,
    FILEGROWTH = 50MB
)
LOG ON (
    NAME = 'MiApp_Log',
    FILENAME = 'C:\Data\MiApp_log.ldf',
    SIZE = 50MB,
    FILEGROWTH = 25MB
);

-- Ver los archivos fisicos de una base de datos existente
SELECT name, physical_name, size * 8 / 1024 AS size_mb, growth
FROM sys.master_files WHERE database_id = DB_ID('MiApp');
```

## Como se usa en un proyecto real

1. Se configura `FILEGROWTH` (crecimiento automatico) con un incremento fijo razonable en vez del porcentaje por defecto, evitando crecimientos descontrolados en bases de datos que ya son grandes.
2. Se separan `.mdf` y `.ldf` en discos fisicos distintos en servidores de produccion de alto rendimiento, distribuyendo I/O de datos y de log por separado.
3. Se monitorea el tamaño y crecimiento de estos archivos como parte del mantenimiento regular (ver capitulo de Administracion), similar al monitoreo de `/var` ya visto en el libro de Linux para Postgres.
4. Se dimensiona el tamaño inicial de forma realista segun el volumen esperado, evitando crecimientos automaticos frecuentes que fragmentan el archivo fisico.
5. Se conecta esto con el capitulo de Backup del libro de PostgreSQL, entendiendo que ambos motores necesitan estrategias de respaldo, aunque con mecanismos fisicos distintos por debajo.

## Criterio junior

Un junior debe poder crear una base de datos especificando la ubicacion de sus archivos `.mdf`/`.ldf` basicos.

## Criterio mid

Un mid debe configurar `FILEGROWTH` apropiadamente y monitorear el tamaño de estos archivos como parte de mantenimiento regular.

## Criterio senior

Un senior debe diseñar la distribucion fisica de archivos de bases de datos criticas (separando datos y log en discos distintos) para optimizar I/O en servidores de produccion de alto rendimiento.

## Errores comunes

- Dejar `FILEGROWTH` en porcentaje por defecto en bases de datos grandes, generando crecimientos descontrolados con el tiempo.
- No monitorear el tamaño de `.mdf`/`.ldf`, descubriendo un disco lleno solo cuando ya causo un incidente.
- No separar datos y log en discos fisicos distintos en servidores de produccion de alto rendimiento, perdiendo oportunidad de optimizacion de I/O.
- Dimensionar el tamaño inicial demasiado conservador, forzando crecimientos automaticos frecuentes que fragmentan el archivo.

## Practica

- [ ] Crea una base de datos especificando ubicacion y tamaño de sus archivos `.mdf`/`.ldf`.
- [ ] Consulta `sys.master_files` para ver el tamaño real y configuracion de crecimiento de una base de datos.
- [ ] Ajusta `FILEGROWTH` a un incremento fijo razonable.
- [ ] Investiga como distribuirias `.mdf` y `.ldf` en discos distintos para un servidor de produccion.
- [ ] Compara esta estructura de archivos con la de Postgres (ver libro complementario).

## Preguntas de repaso

- Que contiene el archivo `.mdf` y que contiene el `.ldf`?
- Para que sirven los archivos `.ndf` opcionales?
- Por que separar datos y log en discos fisicos distintos mejora el rendimiento en servidores de alto trafico?
- Que riesgo tiene dejar `FILEGROWTH` en porcentaje por defecto en una base de datos grande?
