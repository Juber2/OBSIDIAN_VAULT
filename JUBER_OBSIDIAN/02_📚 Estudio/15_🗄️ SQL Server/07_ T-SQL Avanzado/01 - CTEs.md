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

# 01 - CTEs

## Que vas a aprender

En este capitulo vas a dominar **Common Table Expressions** (`WITH nombre AS (...)`) en T-SQL, identicas al estandar SQL ya disponible en Postgres, incluyendo **CTEs recursivas** para consultas jerarquicas.

## Explicacion

Una CTE es una consulta con nombre temporal, valida solo dentro de la sentencia que la define, util para descomponer consultas complejas en partes legibles o para evitar repetir la misma subconsulta multiples veces. Esto es sintacticamente identico entre T-SQL y Postgres, siendo una de las partes mas portables del SQL moderno.

Las **CTEs recursivas** (`WITH cte AS (SELECT ... UNION ALL SELECT ... FROM cte ...)`) resuelven el caso clasico de datos jerarquicos: un organigrama de empleados con jefes, una estructura de categorias con subcategorias, o un arbol de comentarios anidados. La CTE se "referencia a si misma" en la parte recursiva, acumulando resultados nivel por nivel hasta que la condicion de parada (la ausencia de mas filas que coincidan) detiene la recursion.

## Modelo mental

Pensa una CTE simple como darle un apodo temporal a una subconsulta compleja para poder referirse a ella varias veces sin reescribirla. Una CTE recursiva es como pedirle a alguien que suba un nivel en un organigrama, anote a esa persona, y repita la misma pregunta ("quien es tu jefe?") sobre el resultado anterior, una y otra vez, hasta llegar al presidente de la empresa que ya no tiene jefe (la condicion de parada).

## Ejemplo base

```sql
-- CTE simple
WITH VentasPorMes AS (
    SELECT MONTH(CreadoEn) AS Mes, SUM(Total) AS Total
    FROM Pedidos
    GROUP BY MONTH(CreadoEn)
)
SELECT * FROM VentasPorMes WHERE Total > 10000;

-- CTE recursiva: jerarquia de empleados
WITH Jerarquia AS (
    SELECT Id, Nombre, JefeId, 0 AS Nivel
    FROM Empleados WHERE JefeId IS NULL   -- caso base: sin jefe (presidente)

    UNION ALL

    SELECT e.Id, e.Nombre, e.JefeId, j.Nivel + 1
    FROM Empleados e
    INNER JOIN Jerarquia j ON e.JefeId = j.Id   -- parte recursiva
)
SELECT * FROM Jerarquia ORDER BY Nivel;
```

## Como se usa en un proyecto real

1. Se usan CTEs simples para descomponer consultas complejas en pasos legibles, mejorando mantenibilidad frente a subconsultas anidadas profundamente.
2. Se usan CTEs recursivas para cualquier estructura jerarquica del dominio (organigramas, categorias anidadas, estructuras de permisos con herencia).
3. Se aplica exactamente el mismo conocimiento ya transferible de Postgres, dado que la sintaxis de CTEs es prácticamente identica entre ambos motores.
4. Se conecta esto con LINQ de Entity Framework Core, donde ciertas consultas complejas se benefician de escribirse como SQL crudo con CTE cuando LINQ resultaria menos legible.
5. Se documenta cualquier CTE recursiva compleja, explicando la logica de caso base y caso recursivo para futuros mantenedores.

## Criterio junior

Un junior debe poder escribir una CTE simple para descomponer una consulta compleja.

## Criterio mid

Un mid debe escribir una CTE recursiva para una estructura jerarquica real del dominio.

## Criterio senior

Un senior debe decidir cuando una CTE recursiva es apropiada versus alternativas (como Closure Table, un patron de modelado distinto para jerarquias).

## Errores comunes

- No definir correctamente el caso base de una CTE recursiva, generando recursion infinita o resultados incorrectos.
- Usar subconsultas anidadas profundamente en vez de CTEs, sacrificando legibilidad sin necesidad.
- No considerar el limite de recursion por defecto de SQL Server (`MAXRECURSION`, 100 por defecto) para jerarquias muy profundas.
- No aprovechar que este conocimiento es directamente transferible desde Postgres.

## Practica

- [ ] Escribe una CTE simple que descomponga una consulta compleja en un paso legible.
- [ ] Escribe una CTE recursiva para una estructura jerarquica real (organigrama, categorias).
- [ ] Investiga como ajustarias `MAXRECURSION` para una jerarquia muy profunda.
- [ ] Compara la sintaxis de CTEs de T-SQL con la ya usada en Postgres.
- [ ] Identifica una estructura jerarquica real de tu proyecto que se beneficiaria de una CTE recursiva.

## Preguntas de repaso

- Que ventaja de legibilidad da una CTE sobre subconsultas anidadas?
- Como se compone una CTE recursiva (caso base y caso recursivo)?
- Que limite por defecto tiene la recursion en SQL Server y como se ajusta?
- Por que este conocimiento es directamente transferible desde Postgres?
