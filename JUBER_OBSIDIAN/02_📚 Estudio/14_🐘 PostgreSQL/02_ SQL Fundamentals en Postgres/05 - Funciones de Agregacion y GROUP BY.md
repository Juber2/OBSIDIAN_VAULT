---
tags:
  - PostgreSQL
  - SQL
  - Postgres-SQL
Creado: 2026-07-29
Relacionado:
  - SQL Fundamentals en Postgres
  - POSTGRESQL
---

# 05 - Funciones de Agregacion y GROUP BY

## Que vas a aprender

En este capitulo vas a dominar funciones de agregacion (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) combinadas con `GROUP BY` y `HAVING`, la base de cualquier reporte o dashboard construido sobre datos relacionales.

## Explicacion

Las funciones de agregacion resumen multiples filas en un unico valor: `COUNT(*)` cuenta filas, `SUM(columna)` suma valores, `AVG` promedia, `MIN`/`MAX` encuentran extremos. `GROUP BY columna` agrupa filas que comparten el mismo valor en esa columna, aplicando la agregacion **por grupo** en vez de sobre toda la tabla: por ejemplo, `SUM(total) GROUP BY usuario_id` da el total gastado por cada usuario, no un unico total general.

`HAVING` filtra **despues** de la agregacion (por ejemplo, "usuarios con mas de 5 pedidos"), a diferencia de `WHERE` que filtra **antes** de agrupar (filas individuales). Un error de sintaxis comun es intentar usar `WHERE` para filtrar sobre un resultado agregado (`WHERE COUNT(*) > 5`), lo cual falla porque en el momento en que `WHERE` se evalua, la agregacion todavia no existe; `HAVING` es la clausula correcta para ese caso.

## Modelo mental

Pensa `GROUP BY` como organizar una pila de recibos individuales en montones separados por cliente, y la funcion de agregacion como sumar el total de cada monton por separado. `WHERE` decide que recibos individuales entran a la pila desde el principio; `HAVING` decide, despues de haber armado y sumado cada monton, cuales montones (ya sumados) son lo suficientemente grandes como para mostrar en el reporte final.

## Ejemplo base

```sql
SELECT usuario_id, COUNT(*) AS cantidad_pedidos, SUM(total) AS total_gastado
FROM pedidos
WHERE creado_en >= '2026-01-01'          -- filtra filas ANTES de agrupar
GROUP BY usuario_id
HAVING COUNT(*) > 5                       -- filtra grupos DESPUES de agregar
ORDER BY total_gastado DESC;

SELECT categoria, AVG(precio), MIN(precio), MAX(precio)
FROM productos
GROUP BY categoria;
```

## Como se usa en un proyecto real

1. Se construyen reportes y dashboards combinando agregacion con `GROUP BY`, base tecnica de cualquier metrica de negocio agregada.
2. Se usa `WHERE` para reducir el conjunto de filas antes de agrupar (mas eficiente, menos datos que agregar) y `HAVING` solo para condiciones que realmente dependen del resultado agregado.
3. Se combinan agregaciones con `LEFT JOIN` (ya visto en el capitulo anterior) para incluir grupos sin datos (por ejemplo, "categorias sin ningun producto vendido, mostrando 0").
4. Se conecta esto con las queries de agregacion que se construirian en un `QueryBuilder` de TypeORM cuando `find()` simple no alcanza.
5. Se revisa el plan de ejecucion de agregaciones sobre tablas grandes, dado que pueden requerir indices especificos para performance aceptable.

## Criterio junior

Un junior debe poder escribir una consulta con `COUNT`/`SUM`/`AVG` combinada con `GROUP BY` basico.

## Criterio mid

Un mid debe usar `HAVING` correctamente para filtrar sobre resultados agregados, distinguiendolo de `WHERE`.

## Criterio senior

Un senior debe optimizar consultas de agregacion sobre datasets grandes, y diseñar reportes complejos combinando agregacion, joins, y subconsultas de forma eficiente.

## Errores comunes

- Intentar usar `WHERE` para filtrar sobre un resultado de agregacion, obteniendo un error de sintaxis por no entender el orden de evaluacion de las clausulas.
- Poner en `HAVING` una condicion que en realidad podria estar en `WHERE`, agregando y filtrando innecesariamente mas datos de los necesarios.
- Olvidar incluir en `GROUP BY` todas las columnas no agregadas del `SELECT`, generando un error o un resultado ambiguo segun el motor.
- No considerar el impacto de performance de una agregacion sobre una tabla muy grande sin indices apropiados.

## Practica

- [ ] Escribe una consulta que cuente y sume valores agrupados por una columna relevante.
- [ ] Agrega una condicion `HAVING` que filtre sobre el resultado agregado.
- [ ] Provoca el error de intentar usar `WHERE` con una funcion de agregacion y corrigelo con `HAVING`.
- [ ] Combina `GROUP BY` con un `LEFT JOIN` para incluir grupos sin datos relacionados.
- [ ] Anota un reporte real de tu trabajo que podrias construir con estas herramientas.

## Preguntas de repaso

- Que diferencia hay entre `WHERE` y `HAVING` en cuanto a cuando filtran los datos?
- Por que `WHERE COUNT(*) > 5` genera un error de sintaxis?
- Como combinarias `GROUP BY` con `LEFT JOIN` para incluir grupos sin datos relacionados?
- Que consideracion de performance tendrias al agregar datos de una tabla muy grande?
