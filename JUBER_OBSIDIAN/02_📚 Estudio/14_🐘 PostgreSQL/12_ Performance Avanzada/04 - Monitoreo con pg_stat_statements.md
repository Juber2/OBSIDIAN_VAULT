---
tags:
  - PostgreSQL
  - Performance
  - Postgres-Performance-Avanzada
Creado: 2026-07-29
Relacionado:
  - Performance Avanzada
  - POSTGRESQL
---

# 04 - Monitoreo con pg_stat_statements

## Que vas a aprender

En este capitulo vas a profundizar en `pg_stat_statements` (ya instalado en el capitulo de Extensiones), la herramienta central de monitoreo continuo de performance de Postgres en produccion, aprendiendo a interpretar sus metricas y construir un proceso regular de revision.

## Explicacion

`pg_stat_statements` agrega estadisticas de **cada consulta unica** ejecutada (normalizando parametros, agrupando `WHERE id = 1` y `WHERE id = 2` como la misma "forma" de consulta): numero de ejecuciones (`calls`), tiempo total acumulado (`total_exec_time`), tiempo promedio (`mean_exec_time`), y filas devueltas. Esto revela dos tipos de problemas distintos: consultas **individualmente lentas** (alto `mean_exec_time`, pocas `calls`) que se beneficiarian de optimizacion con `EXPLAIN ANALYZE` (ver capitulo de Indices), y consultas **agregadamente costosas** (muchas `calls`, cada una rapida, pero el total acumulado es alto), que se beneficiarian mas de reducir la frecuencia de ejecucion (cache, ver tema anterior) que de optimizar la query en si.

Un proceso de revision regular tipico es consultar `pg_stat_statements` ordenado por `total_exec_time` (revela el mayor impacto agregado real sobre el sistema, no solo la consulta "mas lenta" individualmente) como parte de una rutina periodica de mantenimiento, similar al espiritu de revision continua ya visto en el capitulo de Monitoreo del libro de Docker.

## Modelo mental

Pensa `pg_stat_statements` como un registro contable detallado de todas las transacciones de un negocio: no solo te dice cual fue la transaccion individual mas cara (una consulta ocasionalmente lenta), sino tambien cual categoria de gasto, sumada a lo largo de todas sus repeticiones pequeñas, termina costando mas en total al final del mes (consultas frecuentes pero individualmente baratas).

## Ejemplo base

```sql
-- Top 10 por tiempo TOTAL acumulado: el mayor impacto real sobre el sistema
SELECT query, calls, total_exec_time, mean_exec_time, rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC LIMIT 10;

-- Top 10 por tiempo PROMEDIO: consultas individualmente lentas, candidatas a EXPLAIN ANALYZE
SELECT query, calls, mean_exec_time
FROM pg_stat_statements
WHERE calls > 10   -- descarta consultas ejecutadas muy pocas veces
ORDER BY mean_exec_time DESC LIMIT 10;

-- Reiniciar estadisticas despues de aplicar una optimizacion, para medir el efecto real
SELECT pg_stat_statements_reset();
```

## Como se usa en un proyecto real

1. Se revisa `pg_stat_statements` ordenado por `total_exec_time` regularmente (semanal, por ejemplo) como parte de un proceso de mantenimiento de performance continuo.
2. Se distingue entre consultas individualmente lentas (optimizables con indices, ver capitulo de Indices) y consultas agregadamente costosas por alta frecuencia (candidatas a cache, ver tema anterior).
3. Se usa `pg_stat_statements_reset()` antes de aplicar una optimizacion, para medir el efecto real de ese cambio especifico sin ruido de datos historicos previos.
4. Se conecta esto directamente con el capitulo de Profiling del libro de NestJS, aplicando el mismo principio de "medir antes de optimizar" con evidencia real en vez de suposiciones.
5. Se integra la revision de `pg_stat_statements` con el resto del stack de observabilidad del sistema (ver capitulo de Monitoreo del libro de Docker), correlacionando picos de latencia de aplicacion con queries especificas de base de datos.

## Criterio junior

Un junior debe poder consultar `pg_stat_statements` ordenado por tiempo total y por tiempo promedio, entendiendo la diferencia entre ambos.

## Criterio mid

Un mid debe distinguir entre consultas individualmente lentas y agregadamente costosas, aplicando la solucion apropiada a cada tipo (optimizacion vs cache).

## Criterio senior

Un senior debe establecer un proceso regular de revision de `pg_stat_statements` como parte de la cultura de performance del equipo, integrandolo con el resto del stack de observabilidad.

## Errores comunes

- Optimizar solo la consulta "mas lenta" individualmente sin revisar tambien las agregadamente costosas por alta frecuencia, perdiendo la oportunidad de mayor impacto real.
- No resetear estadisticas antes de medir el efecto de una optimizacion, mezclando datos historicos con el resultado del cambio reciente.
- No tener un proceso regular de revision, descubriendo problemas de performance solo cuando ya afectan visiblemente a usuarios.
- No correlacionar `pg_stat_statements` con el resto del stack de observabilidad de la aplicacion, perdiendo contexto completo de incidentes.

## Practica

- [ ] Consulta `pg_stat_statements` ordenado por `total_exec_time` en un sistema con datos reales o de practica.
- [ ] Identifica al menos una consulta individualmente lenta y otra agregadamente costosa por frecuencia.
- [ ] Aplica una optimizacion (indice o cache, segun corresponda) a cada una.
- [ ] Resetea las estadisticas y mide el efecto real de cada optimizacion.
- [ ] Diseña un proceso regular de revision de `pg_stat_statements` para tu equipo.

## Preguntas de repaso

- Que diferencia hay entre ordenar `pg_stat_statements` por `total_exec_time` versus por `mean_exec_time`?
- Que tipo de solucion es apropiada para una consulta individualmente lenta versus una agregadamente costosa por frecuencia?
- Por que resetear estadisticas antes de medir una optimizacion da una medicion mas confiable?
- Como integrarias la revision de `pg_stat_statements` con el resto del stack de observabilidad del sistema?
