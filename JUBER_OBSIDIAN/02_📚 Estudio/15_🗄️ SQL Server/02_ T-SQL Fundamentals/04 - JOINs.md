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

# 04 - JOINs

## Que vas a aprender

En este capitulo vas a dominar `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN` en T-SQL, con la misma logica fundamental ya vista en el libro de Postgres, y la trampa identica de condiciones en `ON` versus `WHERE` con `LEFT JOIN`.

## Explicacion

Los tipos de JOIN en T-SQL son **semanticamente identicos** a los de Postgres ya estudiados en profundidad en ese libro: `INNER JOIN` solo filas con coincidencia en ambas tablas, `LEFT JOIN` todas las filas de la tabla izquierda con `NULL` donde no hay coincidencia, `RIGHT JOIN` el equivalente invertido, `FULL OUTER JOIN` combina ambos. La sintaxis es practicamente identica entre ambos motores, siendo uno de los aspectos de SQL mas portable entre sistemas relacionales.

La misma trampa exacta ya explicada en el libro de Postgres aplica sin cambios: una condicion adicional en `WHERE` (en vez de en `ON`) puede convertir un `LEFT JOIN` en el equivalente de un `INNER JOIN`, filtrando accidentalmente las filas sin coincidencia que se querian preservar.

## Modelo mental

El modelo mental es exactamente el mismo ya establecido en el libro de Postgres: `INNER JOIN` es la lista de parejas de baile con pareja confirmada, `LEFT JOIN` es la lista completa con "sin pareja" anotado para quien no baila con nadie del otro lado. Esta metafora aplica sin cambios porque la logica relacional del JOIN es un concepto del algebra relacional, independiente del motor especifico que la implementa.

## Ejemplo base

```sql
-- INNER JOIN: solo usuarios que SI tienen pedidos
SELECT u.Nombre, p.Total
FROM Usuarios u
INNER JOIN Pedidos p ON p.UsuarioId = u.Id;

-- LEFT JOIN: TODOS los usuarios, con NULL en pedidos si no tienen ninguno
SELECT u.Nombre, p.Total
FROM Usuarios u
LEFT JOIN Pedidos p ON p.UsuarioId = u.Id;

-- Misma trampa que en Postgres: condicion en WHERE anula el LEFT JOIN
SELECT u.Nombre, p.Total
FROM Usuarios u
LEFT JOIN Pedidos p ON p.UsuarioId = u.Id
WHERE p.Total > 100;   -- excluye usuarios sin pedidos, actua como INNER JOIN

-- Correcto: condicion adicional en el ON
SELECT u.Nombre, p.Total
FROM Usuarios u
LEFT JOIN Pedidos p ON p.UsuarioId = u.Id AND p.Total > 100;
```

## Como se usa en un proyecto real

1. Se aplica exactamente el mismo criterio ya dominado en Postgres para elegir el tipo de JOIN correcto segun la necesidad real de negocio.
2. Se ubica cuidadosamente cualquier condicion adicional en `ON` versus `WHERE` en un `LEFT JOIN`, con el mismo riesgo y solucion ya vistos en el libro complementario.
3. Se conecta esto directamente con las relaciones de Entity Framework Core (ver capitulo de SQL Server con .NET), donde LINQ genera este mismo SQL internamente al usar `.Include()`.
4. Se revisa el plan de ejecucion (ver capitulo de Indices) de joins complejos para verificar indices apropiados, el mismo proceso ya aplicado en Postgres.
5. Se aprovecha que esta habilidad es directamente transferible entre ambos motores, dado que el algebra relacional del JOIN es identica.

## Criterio junior

Un junior debe poder escribir un `INNER JOIN` y un `LEFT JOIN` basico en T-SQL, aplicando lo ya aprendido en Postgres.

## Criterio mid

Un mid debe evitar la trampa de condiciones en `WHERE` versus `ON` en un `LEFT JOIN`, exactamente el mismo cuidado ya practicado en Postgres.

## Criterio senior

Un senior debe optimizar joins complejos verificando planes de ejecucion, transfiriendo directamente el criterio ya desarrollado en el libro de Postgres a T-SQL.

## Errores comunes

- Repetir la misma trampa de condicion en `WHERE` versus `ON` que anula un `LEFT JOIN`, el error identico ya visto en Postgres.
- No revisar el plan de ejecucion de joins complejos, dejando pasar queries lentas por falta de indices.
- Asumir diferencias de sintaxis de JOIN entre T-SQL y Postgres que en realidad no existen, dado que esta parte del SQL es muy portable.
- No aprovechar el conocimiento ya solido de JOINs de Postgres al trabajar en T-SQL, tratandolo como si fuera completamente nuevo.

## Practica

- [ ] Escribe un `INNER JOIN` y un `LEFT JOIN` en T-SQL, comparando la sintaxis con Postgres.
- [ ] Provoca la trampa de condicion en `WHERE` versus `ON` y corrigela.
- [ ] Escribe un `FULL OUTER JOIN` y explica que filas adicionales aparecen.
- [ ] Conecta un `LEFT JOIN` con el `.Include()` equivalente de Entity Framework Core.
- [ ] Revisa el plan de ejecucion de un join complejo en SQL Server (Azure Data Studio o SSMS).

## Preguntas de repaso

- Que diferencia (o similitud) hay entre la sintaxis de JOIN de T-SQL y de Postgres?
- Por que la misma trampa de condicion en `WHERE` versus `ON` aplica identicamente en ambos motores?
- Como se relaciona un `LEFT JOIN` de T-SQL con `.Include()` de Entity Framework Core?
- Que revisarias en el plan de ejecucion de un join complejo en SQL Server?
