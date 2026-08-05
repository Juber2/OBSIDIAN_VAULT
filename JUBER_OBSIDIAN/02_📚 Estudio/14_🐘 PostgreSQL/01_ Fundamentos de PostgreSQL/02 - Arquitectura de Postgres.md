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

# 02 - Arquitectura de Postgres

## Que vas a aprender

En este capitulo vas a entender la arquitectura interna de Postgres: el modelo de **procesos** (no threads, a diferencia de otros motores), la gestion de **memoria**, y una introduccion a **MVCC**, el mecanismo que sustenta la concurrencia de Postgres (profundizado en el capitulo de Transacciones).

## Explicacion

Postgres usa una arquitectura de **multiples procesos**: cada conexion de cliente obtiene su propio proceso backend dedicado en el sistema operativo (no un hilo dentro de un unico proceso), coordinado por un proceso `postmaster` central. Esto da aislamiento robusto entre conexiones (un backend que falla no tumba a los demas) a costa de mayor overhead de memoria por conexion, razon por la cual el pooling de conexiones (ver capitulo de Performance) es tan relevante en Postgres.

La memoria se divide en **memoria compartida** (`shared_buffers`, el cache de paginas de datos compartido entre todos los backends) y **memoria de trabajo** por conexion (`work_mem`, usada para ordenamientos y hash joins). **MVCC** (Multi-Version Concurrency Control) es el mecanismo que permite que lectores y escritores no se bloqueen mutuamente: en vez de bloquear una fila al leerla, Postgres mantiene multiples versiones de cada fila y cada transaccion ve la version correspondiente a su momento de inicio.

## Modelo mental

Pensa cada conexion a Postgres como un empleado con su propio escritorio independiente (proceso backend), en vez de varios empleados compartiendo un unico escritorio por turnos (threads). MVCC es como si, en vez de que alguien tenga que esperar a que otro termine de escribir un documento para poder leerlo, cada lector simplemente recibe una fotocopia del documento tal como estaba en el momento en que empezo a leer, sin bloquear al que sigue escribiendo la version mas nueva.

## Ejemplo base

```sql
SHOW shared_buffers;      -- memoria compartida de cache de paginas
SHOW work_mem;             -- memoria de trabajo por operacion de ordenamiento/hash
SHOW max_connections;       -- limite de conexiones (y por lo tanto de procesos backend)

SELECT pid, usename, state, query FROM pg_stat_activity;
-- muestra cada backend activo como un proceso separado con su propia query
```

```bash
ps aux | grep postgres      -- desde el sistema operativo, cada conexion es un proceso visible
```

## Como se usa en un proyecto real

1. Se dimensiona `max_connections` con cuidado, entendiendo que cada conexion adicional es un proceso completo con su propio overhead de memoria, no un hilo liviano.
2. Se usa connection pooling (PgBouncer, o el pool interno del driver de NestJS/TypeORM) para evitar agotar conexiones bajo carga, en vez de simplemente subir `max_connections` sin limite.
3. Se ajusta `shared_buffers` segun la memoria disponible del servidor, tipicamente entre 25-40% de la RAM total como punto de partida.
4. Se usa `pg_stat_activity` para inspeccionar que backends estan activos y que query esta corriendo cada uno, util para diagnosticar conexiones colgadas.
5. Se conecta este conocimiento con el capitulo de Concurrencia: entender MVCC aca es la base para entender niveles de aislamiento mas adelante.

## Criterio junior

Un junior debe entender que cada conexion a Postgres es un proceso del sistema operativo, y poder ver conexiones activas con `pg_stat_activity`.

## Criterio mid

Un mid debe dimensionar `max_connections`, `shared_buffers` y `work_mem` con criterio basico, y reconocer la necesidad de connection pooling.

## Criterio senior

Un senior debe diseñar la configuracion de memoria y conexiones de servidores Postgres de produccion, balanceando el overhead de procesos por conexion contra la necesidad de pooling en la capa de aplicacion.

## Errores comunes

- Subir `max_connections` indefinidamente para "resolver" errores de conexiones agotadas, sin considerar el overhead de memoria de cada proceso adicional.
- No usar connection pooling, dejando que cada instancia de la aplicacion abra demasiadas conexiones directas al motor.
- Configurar `shared_buffers` demasiado bajo (el default de muchas instalaciones es conservador) sin ajustarlo a la memoria real disponible del servidor.
- No revisar `pg_stat_activity` al diagnosticar un servidor con muchas conexiones aparentemente colgadas.

## Practica

- [ ] Revisa `max_connections`, `shared_buffers` y `work_mem` de tu instalacion con `SHOW`.
- [ ] Verifica en el sistema operativo que cada conexion activa es un proceso separado con `ps aux | grep postgres`.
- [ ] Consulta `pg_stat_activity` para ver las conexiones y queries activas actualmente.
- [ ] Investiga como dimensionarias `shared_buffers` para un servidor con una cantidad de RAM especifica.
- [ ] Anota por que connection pooling es especialmente relevante en Postgres comparado con motores basados en threads.

## Preguntas de repaso

- Por que cada conexion a Postgres es un proceso completo del sistema operativo, no un hilo liviano?
- Que diferencia hay entre `shared_buffers` y `work_mem`?
- Que problema resuelve MVCC entre lectores y escritores concurrentes?
- Por que el connection pooling es particularmente importante en la arquitectura de procesos de Postgres?
