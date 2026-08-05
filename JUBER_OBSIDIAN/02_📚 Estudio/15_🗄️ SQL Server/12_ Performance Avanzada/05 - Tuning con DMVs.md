---
tags:
  - SQLServer
  - PerformanceAvanzada
  - SQLServer-Performance
Creado: 2026-07-29
Relacionado:
  - Performance Avanzada
  - SQL SERVER
---

# 05 - Tuning con DMVs

## Que vas a cerrar

En este capitulo vas a cerrar el capitulo aprendiendo a usar **Dynamic Management Views (DMVs)** de forma sistematica para diagnosticar performance de SQL Server, equivalente directo a `pg_stat_statements`/`pg_stat_activity` ya dominados en profundidad en el libro de Postgres.

## Explicacion

Las DMVs son vistas de sistema que exponen el estado interno en tiempo real del motor de SQL Server, la fuente principal de diagnostico objetivo en vez de depender de sospechas. `sys.dm_exec_requests` muestra las queries actualmente en ejecucion (equivalente a `pg_stat_activity`), `sys.dm_os_wait_stats` acumula estadisticas de en que estan esperando las sesiones a nivel de toda la instancia (identificando si el cuello de botella real es I/O, locking, CPU, u otro tipo de espera), y `sys.dm_db_index_usage_stats` muestra que indices se usan realmente y cuales nunca se tocan (identificando indices candidatos a eliminar, ya mencionado en el capitulo de indices).

El flujo de tuning sistematico con DMVs sigue el mismo principio ya establecido en el libro de Postgres: nunca optimizar por sospecha, siempre medir primero con datos objetivos (que query consume mas recursos, en que esta esperando realmente el sistema, que indices son candidatos a eliminar o agregar) antes de aplicar cualquier cambio, y volver a medir despues para confirmar el impacto real.

## Modelo mental

Pensa las DMVs exactamente igual que `pg_stat_statements`/`pg_stat_activity` ya dominados: son el tablero de instrumentos del motor de base de datos, mostrando en tiempo real que esta pasando realmente adentro, en vez de adivinar basandose en sintomas externos. El mismo habito de "medir antes de optimizar" aplica identicamente, solo cambian los nombres de las vistas de sistema consultadas.

## Ejemplo base

```sql
-- Queries actualmente en ejecucion (equivalente a pg_stat_activity)
SELECT r.session_id, r.status, r.wait_type, t.text AS query_text
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) t
WHERE r.session_id != @@SPID;

-- Estadisticas de espera acumuladas a nivel de instancia
SELECT wait_type, wait_time_ms, waiting_tasks_count
FROM sys.dm_os_wait_stats
ORDER BY wait_time_ms DESC;

-- Indices nunca usados (candidatos a eliminar)
SELECT OBJECT_NAME(i.object_id) AS tabla, i.name AS indice
FROM sys.indexes i
LEFT JOIN sys.dm_db_index_usage_stats s
    ON i.object_id = s.object_id AND i.index_id = s.index_id
WHERE s.index_id IS NULL AND i.type_desc = 'NONCLUSTERED';
```

## Como se usa en un proyecto real

1. Se consulta `sys.dm_exec_requests` durante un incidente de performance en produccion para identificar exactamente que queries estan ejecutandose y en que estan esperando en ese momento.
2. Se revisa `sys.dm_os_wait_stats` periodicamente para entender el patron de espera dominante de la instancia (I/O, locking, CPU), guiando donde enfocar el esfuerzo de tuning.
3. Se usa `sys.dm_db_index_usage_stats` para identificar y eliminar indices nunca usados en StaffTrainingPlatform, reduciendo el overhead de mantenimiento innecesario ya discutido en el capitulo de indices.
4. Se combina esta informacion con Query Store (tema anterior) para tener tanto vision en tiempo real (DMVs) como historica (Query Store) del comportamiento de la base de datos.
5. Se documenta cada hallazgo de tuning con la evidencia de DMVs que lo respalda, antes y despues del cambio aplicado, siguiendo el mismo rigor ya establecido con `pg_stat_statements` en Postgres.

## Criterio junior

Un junior debe poder consultar `sys.dm_exec_requests` para ver que queries estan ejecutandose en un momento dado.

## Criterio mid

Un mid debe usar `sys.dm_os_wait_stats` para identificar el tipo de cuello de botella dominante de una instancia.

## Criterio senior

Un senior debe conducir un proceso de tuning sistematico basado en evidencia de DMVs, midiendo antes y despues de cada cambio para confirmar impacto real.

## Errores comunes

- Optimizar por sospecha sin consultar DMVs primero, aplicando cambios que no atacan el cuello de botella real.
- No revisar `sys.dm_os_wait_stats` periodicamente, perdiendo la oportunidad de detectar patrones de espera problematicos antes de que se vuelvan criticos.
- Eliminar un indice basandose en `sys.dm_db_index_usage_stats` sin considerar que las estadisticas se resetean con cada reinicio del servicio, pudiendo mostrar "no usado" incorrectamente tras un reinicio reciente.
- No documentar el estado antes/despues de un cambio de tuning, perdiendo la capacidad de demostrar el impacto real logrado.

## Practica

- [ ] Consulta `sys.dm_exec_requests` mientras ejecutas una query de larga duracion en otra sesion.
- [ ] Revisa `sys.dm_os_wait_stats` e identifica el tipo de espera dominante en tu instancia de practica.
- [ ] Identifica indices nunca usados con `sys.dm_db_index_usage_stats` en una base de datos con datos de prueba.
- [ ] Diseña un proceso de tuning completo: medir con DMVs, aplicar un cambio, medir de nuevo, documentar el impacto.
- [ ] Compara este flujo con el uso de `pg_stat_statements`/`pg_stat_activity` ya dominado en Postgres.

## Preguntas de repaso

- Que informacion expone `sys.dm_exec_requests` que seria util durante un incidente de performance en vivo?
- Como usarias `sys.dm_os_wait_stats` para identificar si el cuello de botella real es I/O o locking?
- Que precaucion debes tener al usar `sys.dm_db_index_usage_stats` para decidir eliminar un indice?
- Como se compara este flujo de tuning con el ya dominado con `pg_stat_statements` en Postgres?

## Cierre del capitulo

Con este tema cierras el capitulo de performance avanzada de SQL Server, habiendo cubierto particionamiento, Query Store, In-Memory OLTP, Columnstore Indexes, y tuning sistematico con DMVs, todas herramientas que aplicas con el mismo rigor de medicion antes de optimizar ya establecido en el libro de Postgres.
