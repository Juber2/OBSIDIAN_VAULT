---
tags:
  - SQLServer
  - TSQL-Avanzado
  - SQLServer-Avanzado
Creado: 2026-07-29
Relacionado:
  - T-SQL Avanzado
  - SQL SERVER
---

# 02 - Window Functions

## Que vas a aprender

En este capitulo vas a dominar **funciones de ventana** (`OVER (PARTITION BY ... ORDER BY ...)`) en T-SQL: `ROW_NUMBER()`, `RANK()`, `LAG()`/`LEAD()`, y agregaciones con ventana, identicas al estandar SQL disponible en Postgres.

## Explicacion

Las funciones de ventana calculan un valor **por fila**, pero considerando un conjunto ("ventana") de filas relacionadas (definida por `PARTITION BY`), sin colapsar el resultado como haria `GROUP BY`. `ROW_NUMBER()` numera filas secuencialmente dentro de cada particion, `RANK()`/`DENSE_RANK()` dan rankings (con o sin saltos en caso de empates), y `LAG()`/`LEAD()` acceden al valor de una fila anterior/siguiente dentro de la misma particion, extremadamente util para calculos de diferencia entre periodos consecutivos.

Un patron muy comun que combina `ROW_NUMBER()` con una CTE (ver tema anterior) es obtener "el registro mas reciente de cada grupo" (por ejemplo, el ultimo pedido de cada cliente), un caso que de otra forma requeriria subconsultas mas complejas.

## Modelo mental

Pensa una funcion de ventana como darle a cada fila una "vista" hacia sus filas vecinas relacionadas (la particion) sin perder su propia identidad individual, a diferencia de `GROUP BY` que fusiona todas las filas de un grupo en una unica fila resumen. `LAG()`/`LEAD()` son como preguntarle a cada fila "que decia la persona justo antes de vos en esta fila" o "justo despues".

## Ejemplo base

```sql
-- Numerar pedidos de cada cliente, del mas reciente al mas antiguo
SELECT Id, ClienteId, Total, CreadoEn,
       ROW_NUMBER() OVER (PARTITION BY ClienteId ORDER BY CreadoEn DESC) AS Numero
FROM Pedidos;

-- Patron comun: solo el pedido MAS RECIENTE de cada cliente
WITH PedidosNumerados AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY ClienteId ORDER BY CreadoEn DESC) AS rn
    FROM Pedidos
)
SELECT * FROM PedidosNumerados WHERE rn = 1;

-- LAG: comparar el total de un pedido con el total del pedido anterior del mismo cliente
SELECT Id, ClienteId, Total,
       LAG(Total) OVER (PARTITION BY ClienteId ORDER BY CreadoEn) AS TotalPedidoAnterior
FROM Pedidos;
```

## Como se usa en un proyecto real

1. Se usa el patron `ROW_NUMBER()` + CTE para obtener "el ultimo registro de cada grupo", una necesidad muy comun en reportes reales.
2. Se usa `LAG()`/`LEAD()` para calculos de diferencia entre periodos consecutivos (crecimiento mes a mes, comparacion con el registro anterior).
3. Se aplica exactamente el mismo conocimiento transferible desde Postgres, dado que las funciones de ventana son parte del estandar SQL implementado de forma muy similar en ambos motores.
4. Se conecta esto con LINQ de Entity Framework Core, donde ciertos calculos de ventana se expresan mejor como SQL crudo cuando LINQ no tiene una traduccion directa y eficiente.
5. Se documenta cualquier consulta compleja con multiples funciones de ventana combinadas, explicando el objetivo del reporte final.

## Criterio junior

Un junior debe poder usar `ROW_NUMBER()` con `PARTITION BY`/`ORDER BY` para numerar filas dentro de grupos.

## Criterio mid

Un mid debe usar el patron `ROW_NUMBER()` + CTE para obtener el ultimo registro de cada grupo, y `LAG()`/`LEAD()` para comparaciones entre periodos.

## Criterio senior

Un senior debe optimizar consultas complejas con multiples funciones de ventana, verificando su plan de ejecucion para volumenes grandes de datos.

## Errores comunes

- Usar `GROUP BY` cuando en realidad se necesitaba una funcion de ventana para preservar el detalle de cada fila individual junto con el calculo agregado.
- No conocer el patron `ROW_NUMBER()` + CTE, resolviendo "el ultimo registro de cada grupo" con subconsultas mas complejas e ineficientes.
- Confundir `RANK()` y `DENSE_RANK()`, obteniendo resultados inesperados en caso de empates.
- No aprovechar que este conocimiento es directamente transferible desde Postgres.

## Practica

- [ ] Usa `ROW_NUMBER()` para numerar filas dentro de particiones.
- [ ] Implementa el patron `ROW_NUMBER()` + CTE para obtener el ultimo registro de cada grupo de un dataset real.
- [ ] Usa `LAG()` para calcular la diferencia entre un valor y el valor anterior de la misma particion.
- [ ] Compara `RANK()` y `DENSE_RANK()` con datos que tienen empates.
- [ ] Compara la sintaxis de funciones de ventana de T-SQL con la ya conocida de Postgres.

## Preguntas de repaso

- Que diferencia hay entre una funcion de ventana y `GROUP BY` en cuanto a preservar el detalle de cada fila?
- Como usarias `ROW_NUMBER()` combinado con una CTE para obtener el ultimo registro de cada grupo?
- Para que sirve `LAG()` y en que casos de reporte es especialmente util?
- Que diferencia hay entre `RANK()` y `DENSE_RANK()` ante empates?
