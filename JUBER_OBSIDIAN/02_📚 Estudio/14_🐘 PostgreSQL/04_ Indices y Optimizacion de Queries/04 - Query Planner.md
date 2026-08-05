---
tags:
  - PostgreSQL
  - Performance
  - Postgres-Indices
Creado: 2026-07-29
Relacionado:
  - Indices y Optimizacion de Queries
  - POSTGRESQL
---

# 04 - Query Planner

## Que vas a aprender

En este capitulo vas a entender como funciona el **query planner** (planificador de consultas) de Postgres por dentro: como decide entre distintas estrategias de ejecucion, el rol de las **estadisticas** de la tabla, y por que a veces "ignora" un indice que existe.

## Explicacion

El planner de Postgres es **basado en costos**: para cada consulta, genera varios planes de ejecucion posibles (usar este indice, usar aquel, hacer un Seq Scan) y estima el costo de cada uno basandose en **estadisticas** sobre la tabla (cuantas filas tiene, distribucion de valores, cuantos valores unicos por columna), eligiendo el plan con menor costo estimado. Estas estadisticas se recolectan con `ANALYZE` (o automaticamente, ver `autovacuum` en el capitulo de Administracion) y quedan desactualizadas si la tabla cambia significativamente sin refrescarlas.

Es un error comun asumir que "si existe un indice, Postgres siempre lo usa": en tablas muy chicas, o cuando una condicion coincide con una gran proporcion de la tabla, un Seq Scan puede ser realmente **mas rapido** que usar el indice (menos operaciones de I/O aleatorio), y el planner correctamente elige esa opcion. Entender esto evita "pelear" contra el planner intentando forzar el uso de un indice que, en ese caso especifico, no aporta beneficio real.

## Modelo mental

Pensa el query planner como un GPS inteligente que calcula varias rutas posibles y elige la de menor tiempo estimado segun el trafico actual (estadisticas), no necesariamente la ruta "mas corta en distancia" (el indice que existe). A veces el camino directo por la avenida principal (Seq Scan) es mas rapido que el atajo por calles internas (Index Scan) si hay poco trafico en la avenida en ese momento especifico.

## Ejemplo base

```sql
ANALYZE pedidos;    -- actualiza estadisticas manualmente

SELECT relname, n_live_tup, last_analyze FROM pg_stat_user_tables WHERE relname = 'pedidos';
-- muestra cuando se actualizaron las estadisticas por ultima vez

-- Forzar temporalmente deshabilitar seq scan para comparar planes (solo para diagnostico, no en produccion)
SET enable_seqscan = off;
EXPLAIN ANALYZE SELECT * FROM pedidos WHERE estado = 'pendiente';
SET enable_seqscan = on;
```

## Como se usa en un proyecto real

1. Se ejecuta `ANALYZE` manualmente despues de cargas masivas de datos (migraciones, importaciones grandes) que cambian significativamente la distribucion de la tabla antes de que `autovacuum` lo haga automaticamente.
2. Se acepta que el planner elija Seq Scan cuando las estadisticas indican que es realmente la opcion mas eficiente, en vez de forzar el uso de un indice sin necesidad real.
3. Se usa `SET enable_seqscan = off` **solo como herramienta de diagnostico temporal** para comparar planes alternativos, nunca como configuracion permanente de produccion.
4. Se revisa `pg_stat_user_tables` para verificar cuando se actualizaron por ultima vez las estadisticas de tablas criticas.
5. Se conecta esto con el capitulo de Administracion, donde `autovacuum` se encarga de mantener estadisticas actualizadas automaticamente en la mayoria de casos.

## Criterio junior

Un junior debe entender que el planner elige planes basandose en estadisticas, no solo en la existencia de un indice.

## Criterio mid

Un mid debe ejecutar `ANALYZE` manualmente despues de cambios masivos de datos, y verificar cuando se actualizaron las estadisticas de una tabla.

## Criterio senior

Un senior debe diagnosticar casos donde el planner elige un plan subóptimo por estadisticas desactualizadas o configuracion incorrecta, y ajustar la configuracion de `autovacuum`/`ANALYZE` para mantener estadisticas frescas en tablas criticas de alto volumen.

## Errores comunes

- Asumir que un indice existente siempre se usa, sin entender que el planner puede elegir correctamente Seq Scan en ciertos casos.
- No ejecutar `ANALYZE` despues de una carga masiva de datos, dejando que el planner tome decisiones basadas en estadisticas obsoletas.
- Dejar `enable_seqscan = off` como configuracion permanente en vez de usarla solo como herramienta de diagnostico temporal.
- No revisar `pg_stat_user_tables` para verificar la frescura de las estadisticas al diagnosticar un plan inesperado.

## Practica

- [ ] Ejecuta `ANALYZE` manualmente en una tabla y verifica el cambio en `pg_stat_user_tables`.
- [ ] Compara el plan de una consulta antes y despues de una carga masiva de datos sin `ANALYZE`.
- [ ] Usa `SET enable_seqscan = off` temporalmente para comparar el plan alternativo, y vuelve a activarlo despues.
- [ ] Identifica un caso donde Seq Scan seria realmente mas eficiente que usar un indice existente.
- [ ] Investiga como se configura `autovacuum` para mantener estadisticas actualizadas automaticamente.

## Preguntas de repaso

- En que basa el query planner su decision de que plan de ejecucion usar?
- Por que a veces un Seq Scan es realmente mas eficiente que usar un indice existente?
- Que hace `ANALYZE` y cuando deberia ejecutarse manualmente?
- Por que `enable_seqscan = off` deberia usarse solo como diagnostico temporal, nunca en produccion permanente?
