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

# 05 - N Mas 1 y Optimizacion de Joins

## Que vas a aprender

En este capitulo vas a cerrar el capitulo aplicando todo lo aprendido al problema de **N+1** (ya visto en el libro de NestJS desde la perspectiva de TypeORM) ahora desde el lado puro de SQL, y a optimizar joins complejos entre multiples tablas.

## Explicacion

El problema de N+1, visto en el libro de NestJS como un problema de ORM, es en el fondo un problema de **cuantas queries SQL se ejecutan**: 1 query para traer N filas, y despues N queries adicionales (una por fila) para traer datos relacionados, en vez de resolverlo con un unico `JOIN` o con una segunda query que traiga todo lo relacionado de una vez (`WHERE id IN (...)`). Ver esto reflejado en logs de SQL reales (activando `log_statement = 'all'` temporalmente, o revisando logs de queries de TypeORM) es la forma mas directa de confirmar si un ORM esta generando N+1 o no.

Para joins complejos entre multiples tablas, el orden en que Postgres decide **unir** las tablas (no necesariamente el orden en que aparecen en el `FROM`) es decidido por el planner segun costos estimados; `EXPLAIN ANALYZE` revela ese orden real de ejecucion. Cuando un join complejo es lento, generalmente el problema es falta de indices en las columnas de union (`ON`), no el join en si mismo.

## Modelo mental

Pensa N+1 como pedirle a un mensajero que primero traiga la lista completa de destinatarios de una campaña (1 query), y despues, en vez de entregar todas las cartas en un unico viaje organizado (JOIN), haga un viaje separado de ida y vuelta por cada destinatario individual (N queries), cuando un unico viaje bien planificado hubiera bastado.

## Ejemplo base

```sql
-- N+1 real en SQL: 1 query para pedidos, despues N queries (una por pedido) para su cliente
SELECT * FROM pedidos;
-- luego, por cada pedido:
SELECT * FROM clientes WHERE id = $1;   -- repetido N veces

-- Correcto: un unico JOIN
SELECT p.*, c.nombre, c.email
FROM pedidos p
JOIN clientes c ON c.id = p.cliente_id;

-- Alternativa: una segunda query que trae TODOS los clientes relacionados de una vez
SELECT * FROM clientes WHERE id IN (1, 2, 3, 4, 5);
```

```sql
-- Revisar el orden real de ejecucion de un join complejo
EXPLAIN ANALYZE
SELECT p.id, c.nombre, pi.producto_id
FROM pedidos p
JOIN clientes c ON c.id = p.cliente_id
JOIN pedido_items pi ON pi.pedido_id = p.id
WHERE c.activo = true;
```

## Como se usa en un proyecto real

1. Se revisan logs de SQL generados por el ORM (TypeORM, ver libro de NestJS) durante desarrollo, contando queries reales ejecutadas por un endpoint para detectar N+1 antes de que llegue a produccion.
2. Se corrige N+1 usando `relations`/`JOIN` explicito en la query principal, o con una segunda query batch (`IN (...)`) cuando un join directo complica demasiado la estructura del resultado.
3. Se indexan las columnas usadas en `ON` de joins complejos y frecuentes, dado que es la causa mas comun de lentitud en joins entre tablas grandes.
4. Se usa `EXPLAIN ANALYZE` para verificar el orden real en que Postgres decide unir tablas en un join de tres o mas tablas, entendiendo que no siempre coincide con el orden del `FROM`.
5. Se conecta esto directamente con el capitulo de N+1 del libro de NestJS, cerrando el circulo entre el problema visto a nivel de ORM y su causa raiz a nivel de SQL puro.

## Criterio junior

Un junior debe poder identificar N+1 contando queries SQL reales generadas por una operacion, y corregirlo con un JOIN simple.

## Criterio mid

Un mid debe indexar apropiadamente columnas de union en joins complejos, y elegir entre JOIN directo o query batch segun la estructura del resultado necesitado.

## Criterio senior

Un senior debe establecer el proceso de revision de queries SQL generadas por el ORM como parte del flujo de desarrollo del equipo, detectando N+1 sistematicamente antes de que llegue a produccion.

## Errores comunes

- No revisar los logs de SQL reales generados por el ORM, dejando pasar N+1 sin detectarlo hasta que aparece como problema de performance en produccion.
- No indexar columnas usadas en `ON` de joins frecuentes, generando lentitud atribuida erroneamente al join en si en vez de a la falta de indice.
- Asumir que el orden de tablas en `FROM` determina el orden real de ejecucion, sin verificar con `EXPLAIN ANALYZE` el plan real elegido por el planner.
- Elegir siempre JOIN directo incluso cuando una query batch con `IN (...)` produciria una estructura de datos mas simple de manejar en el codigo de aplicacion.

## Practica

- [ ] Provoca un caso de N+1 real ejecutando manualmente 1 query mas N queries adicionales, contando el total.
- [ ] Corrige ese caso con un JOIN directo y verifica que ahora es una unica query.
- [ ] Indexa las columnas de union de un join complejo y mide la mejora con `EXPLAIN ANALYZE`.
- [ ] Revisa el orden real de ejecucion de un join de tres tablas con `EXPLAIN ANALYZE`.
- [ ] Conecta este capitulo con el capitulo de N+1 del libro de NestJS, revisando logs de SQL generados por TypeORM en un proyecto real.

## Preguntas de repaso

- Como se ve el problema de N+1 reflejado en queries SQL reales, mas alla de la abstraccion del ORM?
- Cuando conviene corregir N+1 con un JOIN directo versus una query batch con `IN (...)`?
- Que columnas son las mas importantes de indexar en un join complejo?
- Por que el orden de tablas en `FROM` no necesariamente determina el orden real de ejecucion del join?
