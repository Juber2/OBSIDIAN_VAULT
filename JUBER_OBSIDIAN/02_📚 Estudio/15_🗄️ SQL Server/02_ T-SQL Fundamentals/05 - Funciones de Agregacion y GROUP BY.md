---
tags:
  - SQLServer
  - TSQL
  - SQLServer-TSQL
Creado: 2026-07-29
Relacionado:
  - T-SQL Fundamentals
  - SQL SERVER
---

# 05 - Funciones de Agregacion y GROUP BY

## Que vas a aprender

En este capitulo vas a dominar `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` con `GROUP BY`/`HAVING` en T-SQL, identico en logica a lo ya visto en Postgres, y features especificas de T-SQL como `GROUPING SETS`.

## Explicacion

La logica de agregacion en T-SQL es identica a Postgres: `GROUP BY` agrupa filas, las funciones de agregacion resumen cada grupo, `HAVING` filtra despues de agregar mientras `WHERE` filtra antes. T-SQL agrega `GROUPING SETS` (y sus variantes `ROLLUP`/`CUBE`), una extension que permite calcular **multiples niveles de agregacion en una unica consulta** (por ejemplo, totales por categoria, totales por region, y el gran total general, todo en un unico resultado), evitando tener que ejecutar y combinar manualmente varias consultas separadas.

## Modelo mental

Pensa `GROUP BY` simple como organizar recibos en montones por cliente y sumar cada monton por separado (identico a Postgres). `GROUPING SETS` es como pedirle al mismo ejercicio que ademas te de, en el mismo reporte, el subtotal por ciudad, el subtotal por region, y el gran total general, todo calculado de una vez sin tener que repetir la consulta base tres veces con distintos niveles de agrupacion.

## Ejemplo base

```sql
SELECT UsuarioId, COUNT(*) AS CantidadPedidos, SUM(Total) AS TotalGastado
FROM Pedidos
WHERE CreadoEn >= '2026-01-01'
GROUP BY UsuarioId
HAVING COUNT(*) > 5
ORDER BY TotalGastado DESC;

-- GROUPING SETS: multiples niveles de agregacion en una sola consulta
SELECT Categoria, Region, SUM(Total) AS Total
FROM Pedidos p JOIN Productos pr ON pr.Id = p.ProductoId
GROUP BY GROUPING SETS (
    (Categoria, Region),   -- por categoria y region
    (Categoria),             -- solo por categoria
    ()                        -- gran total general
);
```

## Como se usa en un proyecto real

1. Se aplica exactamente la misma logica de agregacion ya dominada en Postgres, sin necesitar reaprender el concepto fundamental.
2. Se usa `GROUPING SETS`/`ROLLUP` para reportes que necesitan multiples niveles de subtotal en una unica consulta, evitando combinar manualmente varias consultas separadas.
3. Se sigue usando `WHERE` para reducir filas antes de agrupar (mas eficiente) y `HAVING` solo para condiciones sobre el resultado agregado.
4. Se conecta esto con las agregaciones de LINQ en Entity Framework Core (ver capitulo de SQL Server con .NET), donde `.GroupBy()` genera SQL equivalente.
5. Se revisa el plan de ejecucion de agregaciones sobre tablas grandes, el mismo proceso ya aplicado en Postgres.

## Criterio junior

Un junior debe poder escribir una consulta con agregacion y `GROUP BY` basico en T-SQL, transfiriendo directamente lo ya aprendido en Postgres.

## Criterio mid

Un mid debe usar `HAVING` correctamente y conocer `GROUPING SETS` como feature especifica de T-SQL para reportes multi-nivel.

## Criterio senior

Un senior debe optimizar reportes complejos con multiples niveles de agregacion usando `GROUPING SETS`/`ROLLUP` en vez de combinar manualmente consultas separadas.

## Errores comunes

- No aprovechar que la logica de agregacion es identica a Postgres, tratandola como concepto nuevo a reaprender desde cero.
- Combinar manualmente varias consultas para obtener distintos niveles de subtotal cuando `GROUPING SETS` lo resolveria en una unica consulta.
- Repetir el error clasico de intentar usar `WHERE` para filtrar sobre un resultado agregado, el mismo error ya visto en Postgres.
- No considerar el impacto de performance de una agregacion multi-nivel sobre una tabla muy grande.

## Practica

- [ ] Escribe una consulta con agregacion y `GROUP BY`/`HAVING`, comparando la sintaxis con Postgres.
- [ ] Escribe una consulta con `GROUPING SETS` que calcule multiples niveles de subtotal.
- [ ] Compara el resultado de `GROUPING SETS` con hacer las mismas agregaciones por separado.
- [ ] Conecta una agregacion T-SQL con el `.GroupBy()` equivalente de LINQ.
- [ ] Diseña un reporte real de tu trabajo que se beneficiaria de `GROUPING SETS`.

## Preguntas de repaso

- Que diferencia (o similitud) hay entre la logica de `GROUP BY`/`HAVING` de T-SQL y Postgres?
- Que problema resuelve `GROUPING SETS` que tendrias que resolver de otra forma sin esa feature?
- Como se relaciona una agregacion T-SQL con `.GroupBy()` de LINQ en Entity Framework Core?
- Por que conviene usar `WHERE` para reducir filas antes de `GROUP BY` en vez de filtrar despues con `HAVING`?
