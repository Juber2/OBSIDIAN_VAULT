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

# 02 - EXPLAIN ANALYZE

## Que vas a aprender

En este capitulo vas a dominar `EXPLAIN` y `EXPLAIN ANALYZE`, ya mencionados de forma introductoria en el libro de NestJS, ahora en profundidad: como leer un plan de ejecucion completo de Postgres, identificando Seq Scan, Index Scan, y el costo real de cada operacion.

## Explicacion

`EXPLAIN consulta` muestra el **plan de ejecucion estimado** que Postgres elegiria, sin ejecutar realmente la consulta. `EXPLAIN ANALYZE consulta` **ejecuta** la consulta de verdad y muestra tiempos reales medidos junto con el plan, siendo la herramienta correcta para diagnostico real (con la salvedad de que ejecuta la consulta, por lo que hay que tener cuidado con `INSERT`/`UPDATE`/`DELETE` reales al usarla, o envolverlos en una transaccion con `ROLLBACK`).

El plan se lee de **adentro hacia afuera** (las operaciones mas internas/profundas se ejecutan primero): un **Seq Scan** (escaneo secuencial de toda la tabla) es costoso en tablas grandes y generalmente indica falta de un indice apropiado; un **Index Scan** o **Index Only Scan** (este ultimo, cuando el indice contiene toda la informacion necesaria sin tocar la tabla) son mucho mas eficientes. El costo estimado (`cost=X..Y`) y las filas estimadas vs las filas reales (`rows=X actual rows=Y`) revelan si las estadisticas del planificador estan desactualizadas (ver `ANALYZE` en el capitulo de Administracion).

## Modelo mental

Pensa `EXPLAIN` como pedirle a un GPS que muestre la ruta que tomaria sin conducir realmente. `EXPLAIN ANALYZE` es conducir la ruta de verdad y medir cuanto tardo realmente cada tramo, comparando con lo que el GPS habia estimado. Un Seq Scan es como recorrer cada casa de una ciudad entera buscando una direccion especifica; un Index Scan es usar el mapa con indice de calles para ir directo a la cuadra correcta.

## Ejemplo base

```sql
EXPLAIN ANALYZE
SELECT * FROM pedidos WHERE cliente_id = 42 ORDER BY creado_en DESC LIMIT 10;

-- Sin indice: Seq Scan on pedidos (cost=0.00..1834.00 rows=50 width=...)
--   Filter: (cliente_id = 42)
--   Rows Removed by Filter: 49950

-- Con indice en (cliente_id, creado_en):
-- Index Scan using idx_pedidos_cliente_fecha on pedidos
--   (cost=0.42..8.44 rows=50 width=...) (actual time=0.02..0.05 rows=48 loops=1)
```

```sql
-- Envolver en transaccion con ROLLBACK para probar EXPLAIN ANALYZE de un UPDATE sin efecto real
BEGIN;
EXPLAIN ANALYZE UPDATE pedidos SET estado = 'procesado' WHERE cliente_id = 42;
ROLLBACK;
```

## Como se usa en un proyecto real

1. Se usa `EXPLAIN ANALYZE` como primer paso ante cualquier query lenta, en vez de adivinar o agregar indices al azar.
2. Se busca especificamente **Seq Scan** en tablas grandes como señal de que falta un indice apropiado para el patron de filtro usado.
3. Se compara `rows` estimado versus `actual rows`: una diferencia grande indica estadisticas desactualizadas, solucionable con `ANALYZE` (ver capitulo de Administracion).
4. Se envuelve `EXPLAIN ANALYZE` de operaciones de escritura en una transaccion con `ROLLBACK`, evitando aplicar el cambio real solo por querer medir su plan.
5. Se conecta esto directamente con el capitulo de N+1 del libro de NestJS, usando `EXPLAIN ANALYZE` para verificar en la practica si una relacion cargada genera multiples queries o una sola eficiente.

## Criterio junior

Un junior debe poder ejecutar `EXPLAIN ANALYZE` sobre una query simple e identificar si usa Seq Scan o Index Scan.

## Criterio mid

Un mid debe leer un plan de ejecucion completo con multiples operaciones anidadas, identificando el cuello de botella real entre varias etapas.

## Criterio senior

Un senior debe usar `EXPLAIN ANALYZE` sistematicamente en revisiones de performance, estableciendo el proceso de optimizacion de queries como parte del flujo de desarrollo del equipo.

## Errores comunes

- Agregar indices "a ciegas" sin haber usado `EXPLAIN ANALYZE` primero para confirmar que realmente falta un indice en ese punto especifico.
- Ejecutar `EXPLAIN ANALYZE` de un `UPDATE`/`DELETE` sin envolverlo en una transaccion con `ROLLBACK`, aplicando el cambio real sin quererlo.
- No comparar filas estimadas versus reales, perdiendo la señal de que las estadisticas del planificador necesitan actualizarse.
- Interpretar un Seq Scan como siempre malo, sin considerar que en tablas muy chicas puede ser mas eficiente que usar un indice.

## Practica

- [ ] Ejecuta `EXPLAIN ANALYZE` sobre una consulta sin indice apropiado y observa el Seq Scan.
- [ ] Crea el indice correspondiente y vuelve a ejecutar, comparando el plan resultante.
- [ ] Ejecuta `EXPLAIN ANALYZE` de un `UPDATE` envuelto en una transaccion con `ROLLBACK`.
- [ ] Identifica una diferencia entre filas estimadas y reales en algun plan que hayas generado.
- [ ] Conecta un caso de N+1 (visto en el libro de NestJS) con lo que revelaria `EXPLAIN ANALYZE` sobre esas queries.

## Preguntas de repaso

- Que diferencia hay entre `EXPLAIN` y `EXPLAIN ANALYZE`?
- Que indica un Seq Scan sobre una tabla grande en un plan de ejecucion?
- Como usarias una transaccion con `ROLLBACK` para medir el plan de un `UPDATE` sin aplicarlo realmente?
- Que revela una gran diferencia entre filas estimadas y reales en un plan de ejecucion?
