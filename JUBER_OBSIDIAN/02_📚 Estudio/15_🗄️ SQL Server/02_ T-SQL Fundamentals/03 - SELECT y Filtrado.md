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

# 03 - SELECT y Filtrado

## Que vas a aprender

En este capitulo vas a dominar `SELECT` con `TOP`, `WHERE`, `ORDER BY`, y `OFFSET`/`FETCH` (el equivalente de `LIMIT`/`OFFSET` de Postgres) en T-SQL, incluyendo el comportamiento especifico de `LIKE` en SQL Server, distinto al de Postgres.

## Explicacion

`SELECT TOP N columnas FROM tabla WHERE condicion` limita resultados con la sintaxis especifica de SQL Server ya vista en el capitulo de Fundamentos. Para paginacion real (con offset), T-SQL usa `OFFSET N ROWS FETCH NEXT M ROWS ONLY`, mas verboso que `LIMIT`/`OFFSET` de Postgres pero funcionalmente equivalente, requiriendo obligatoriamente un `ORDER BY` para que el resultado sea deterministico.

Una diferencia critica: `LIKE` en SQL Server es **case-insensitive por defecto** en la mayoria de configuraciones de collation (a diferencia de Postgres, donde `LIKE` es case-sensitive y se necesita `ILIKE` explicitamente, ver ese libro), aunque esto depende de la **collation** configurada en la base de datos (`SQL_Latin1_General_CP1_CI_AS` es collation case-insensitive comun, mientras que una con `_CS` seria case-sensitive). `IS NULL`/`IS NOT NULL` funcionan identicamente en ambos motores.

## Modelo mental

Pensa la collation de SQL Server como el "acento regional" configurado para toda la base de datos: determina automaticamente si las comparaciones de texto distinguen mayusculas/minusculas o no, sin que cada consulta individual tenga que especificarlo explicitamente como si necesita hacerlo Postgres con `ILIKE`.

## Ejemplo base

```sql
SELECT Nombre, Email FROM Usuarios
WHERE Activo = 1 AND Edad >= 18
ORDER BY CreadoEn DESC
OFFSET 40 ROWS FETCH NEXT 20 ROWS ONLY;

SELECT * FROM Productos WHERE Nombre LIKE '%camisa%';
-- case-insensitive por defecto en la mayoria de collations de SQL Server

SELECT * FROM Usuarios WHERE Telefono IS NULL;   -- identico a Postgres

-- Verificar la collation de la base de datos actual
SELECT DATABASEPROPERTYEX('MiApp', 'Collation');
```

## Como se usa en un proyecto real

1. Se usa `OFFSET`/`FETCH` con `ORDER BY` obligatorio para paginacion, verificando siempre que la columna de orden tiene indice apropiado (ver capitulo de Indices).
2. Se verifica la collation de la base de datos antes de asumir comportamiento case-insensitive de `LIKE`, dado que puede configurarse distinto segun necesidades del proyecto.
3. Se documenta claramente esta diferencia de comportamiento entre `LIKE` de SQL Server y Postgres para el equipo que trabaja con ambos motores.
4. Se evalua paginacion por cursor en vez de `OFFSET` para listados de alto volumen, aplicando el mismo criterio ya visto en el capitulo de SELECT del libro de Postgres.
5. Se conecta esto con LINQ de Entity Framework Core (ver capitulo de SQL Server con .NET), donde `.Take()`/`.Skip()` generan esta misma sintaxis `OFFSET`/`FETCH` automaticamente.

## Criterio junior

Un junior debe poder escribir un `SELECT` con `WHERE`, `ORDER BY` y `TOP`/`OFFSET-FETCH` basico.

## Criterio mid

Un mid debe verificar la collation de una base de datos antes de asumir comportamiento case-sensitive/insensitive de `LIKE`.

## Criterio senior

Un senior debe estandarizar la collation usada en bases de datos del equipo, documentando esa decision y su impacto en comparaciones de texto.

## Errores comunes

- Asumir que `LIKE` en SQL Server siempre es case-insensitive sin verificar la collation real configurada.
- Usar `OFFSET`/`FETCH` sin `ORDER BY`, obteniendo un error de sintaxis (SQL Server lo exige, a diferencia de algunos otros motores mas permisivos).
- Usar `OFFSET` para paginar datasets muy grandes sin considerar el impacto de performance en paginas profundas, el mismo problema ya visto en Postgres.
- No documentar diferencias de comportamiento de `LIKE` entre SQL Server y Postgres para equipos que trabajan con ambos.

## Practica

- [ ] Escribe una consulta con `OFFSET`/`FETCH` para paginacion, verificando que incluye `ORDER BY`.
- [ ] Verifica la collation de una base de datos con `DATABASEPROPERTYEX`.
- [ ] Prueba `LIKE` con texto de distinta capitalizacion y observa el comportamiento segun la collation.
- [ ] Compara la sintaxis de paginacion de T-SQL con `LIMIT`/`OFFSET` de Postgres.
- [ ] Documenta el comportamiento de `LIKE` de tu base de datos actual para referencia del equipo.

## Preguntas de repaso

- Que diferencia de sintaxis hay entre la paginacion de T-SQL (`OFFSET`/`FETCH`) y Postgres (`LIMIT`/`OFFSET`)?
- Por que `LIKE` en SQL Server puede comportarse distinto a `LIKE` en Postgres?
- Que exige T-SQL obligatoriamente al usar `OFFSET`/`FETCH` que otros motores no siempre requieren?
- Como verificarias la collation configurada de una base de datos SQL Server?
