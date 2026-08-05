---
tags:
  - SQLServer
  - Performance
  - SQLServer-Indices
Creado: 2026-07-29
Relacionado:
  - Indices y Optimizacion de Queries
  - SQL SERVER
---

# 05 - Statistics y su Actualizacion

## Que vas a aprender

En este capitulo vas a cerrar el capitulo aprendiendo **Statistics** de SQL Server, el equivalente exacto de lo que `ANALYZE` mantiene en Postgres (ya estudiado en ese libro), y como diagnosticar cuando estan desactualizadas.

## Explicacion

SQL Server mantiene estadisticas sobre la distribucion de valores de columnas y indices, usadas por el Query Optimizer para estimar costos, identico en proposito a las estadisticas de Postgres. Por defecto, `AUTO_UPDATE_STATISTICS` esta activado, actualizando estadisticas automaticamente cuando una porcion suficiente de la tabla cambio (similar en espiritu a `autovacuum`/`ANALYZE` automatico de Postgres), aunque el umbral y comportamiento exacto difieren entre versiones.

Al igual que en Postgres, despues de una carga masiva de datos es buena practica actualizar estadisticas manualmente (`UPDATE STATISTICS`) en vez de esperar al ciclo automatico, para que el optimizer tenga informacion actual inmediatamente. Se puede diagnosticar estadisticas desactualizadas comparando el numero de filas modificadas desde la ultima actualizacion contra el tamaño total de la tabla.

## Modelo mental

El modelo mental es identico al ya establecido con `ANALYZE` en Postgres: las estadisticas son el "censo poblacional" que el optimizador consulta para tomar decisiones, y si ese censo esta desactualizado (la poblacion real cambio significativamente desde el ultimo censo), las decisiones basadas en el pueden ser incorrectas aunque el proceso de decision en si sea correcto.

## Ejemplo base

```sql
UPDATE STATISTICS Pedidos;    -- actualiza estadisticas de toda la tabla, similar a ANALYZE

UPDATE STATISTICS Pedidos IX_Pedidos_ClienteId;   -- solo un indice especifico

-- Ver cuando se actualizaron las estadisticas por ultima vez
SELECT s.name, sp.last_updated, sp.rows, sp.modification_counter
FROM sys.stats s
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) sp
WHERE s.object_id = OBJECT_ID('Pedidos');

-- Verificar si AUTO_UPDATE_STATISTICS esta activo para la base de datos
SELECT name, is_auto_update_stats_on FROM sys.databases WHERE name = 'MiApp';
```

## Como se usa en un proyecto real

1. Se ejecuta `UPDATE STATISTICS` manualmente despues de cargas masivas de datos (migraciones, importaciones), aplicando el mismo principio ya establecido con `ANALYZE` en Postgres.
2. Se verifica `modification_counter` para saber cuantas filas cambiaron desde la ultima actualizacion, comparando contra el tamaño total para estimar si las estadisticas estan significativamente desactualizadas.
3. Se confia en `AUTO_UPDATE_STATISTICS` para la mayoria de tablas con cambios graduales, interviniendo manualmente solo ante cargas masivas puntuales.
4. Se conecta esto directamente con el capitulo de Query Planner de Postgres, aplicando exactamente el mismo criterio de diagnostico ("verificar frescura de estadisticas antes de asumir un plan subóptimo") con la sintaxis especifica de SQL Server.
5. Se combina la actualizacion de estadisticas con la reconstruccion de indices fragmentados (ver capitulo de Administracion) como parte del mantenimiento regular.

## Criterio junior

Un junior debe poder ejecutar `UPDATE STATISTICS` manualmente y verificar cuando se actualizaron por ultima vez.

## Criterio mid

Un mid debe diagnosticar estadisticas desactualizadas comparando `modification_counter` contra el tamaño de la tabla.

## Criterio senior

Un senior debe establecer un proceso de mantenimiento regular de estadisticas para bases de datos criticas, integrandolo con el mantenimiento de indices.

## Errores comunes

- No actualizar estadisticas manualmente despues de una carga masiva de datos, dejando que el optimizer opere con informacion obsoleta hasta el proximo ciclo automatico.
- No verificar `modification_counter` al diagnosticar un plan de ejecucion inesperado, perdiendo la señal de estadisticas desactualizadas.
- Asumir que este concepto es completamente nuevo, sin transferir el criterio ya solido de `ANALYZE` en Postgres.
- No integrar la actualizacion de estadisticas con el mantenimiento regular de indices del sistema.

## Practica

- [ ] Ejecuta `UPDATE STATISTICS` manual en una tabla de practica.
- [ ] Consulta `sys.dm_db_stats_properties` para ver cuando se actualizaron por ultima vez.
- [ ] Simula una carga masiva de datos y compara el plan de ejecucion antes y despues de actualizar estadisticas.
- [ ] Verifica si `AUTO_UPDATE_STATISTICS` esta activo en una base de datos real.
- [ ] Diseña un proceso de mantenimiento regular de estadisticas para un sistema de produccion.

## Preguntas de repaso

- Que rol cumplen las estadisticas de SQL Server, identico en proposito a lo ya visto con `ANALYZE` en Postgres?
- Por que conviene actualizar estadisticas manualmente despues de una carga masiva de datos?
- Como verificarias si las estadisticas de una tabla estan significativamente desactualizadas?
- Con que otra tarea de mantenimiento integrarias la actualizacion regular de estadisticas?
