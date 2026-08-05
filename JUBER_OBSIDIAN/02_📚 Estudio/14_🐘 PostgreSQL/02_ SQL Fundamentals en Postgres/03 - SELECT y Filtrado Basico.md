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

# 03 - SELECT y Filtrado Basico

## Que vas a aprender

En este capitulo vas a dominar `SELECT` y las clausulas de filtrado y ordenamiento: `WHERE`, `ORDER BY`, `LIMIT`/`OFFSET`, y los operadores de comparacion y logicos mas usados, incluyendo particularidades de Postgres como `ILIKE`.

## Explicacion

`SELECT columnas FROM tabla WHERE condicion` es la base de cualquier consulta de lectura. Postgres soporta los operadores estandar (`=`, `!=`, `<`, `>`, `AND`, `OR`, `IN`, `BETWEEN`) mas extensiones utiles: `ILIKE` hace busqueda de texto **case-insensitive** (a diferencia de `LIKE`, que es sensible a mayusculas por defecto en Postgres, distinto del comportamiento de otros motores como SQL Server donde por defecto no lo es), e `IS NULL`/`IS NOT NULL` (nunca `= NULL`, que en SQL siempre evalua a `NULL`, no a verdadero, un error clasico de principiantes).

`ORDER BY` ordena resultados (`ASC` por defecto, `DESC` para descendente), y `LIMIT`/`OFFSET` paginan resultados (ya visto conceptualmente en el capitulo de Paginacion del libro de NestJS), aunque para datasets grandes la paginacion por cursor (basada en el ultimo valor visto, no en offset) escala mejor que `OFFSET`, que se vuelve mas lento a medida que el offset crece.

## Modelo mental

Pensa `WHERE` como un colador que solo deja pasar las filas que cumplen la condicion exacta, `ORDER BY` como ordenar las cartas resultantes de una baraja segun un criterio, y `LIMIT`/`OFFSET` como decir "dame solo las primeras 20 cartas de esa pila ordenada, saltando las primeras 40".

## Ejemplo base

```sql
SELECT nombre, email FROM usuarios
WHERE activo = true AND edad >= 18
ORDER BY creado_en DESC
LIMIT 20 OFFSET 40;

SELECT * FROM productos WHERE nombre ILIKE '%camisa%';    -- busqueda insensible a mayusculas

SELECT * FROM usuarios WHERE telefono IS NULL;              -- correcto
SELECT * FROM usuarios WHERE telefono = NULL;                -- INCORRECTO, nunca coincide

SELECT * FROM productos WHERE categoria IN ('ropa', 'calzado');
```

## Como se usa en un proyecto real

1. Se usa `ILIKE` para busquedas de texto que deben ser insensibles a mayusculas, comun en filtros de busqueda de aplicaciones reales.
2. Se usa siempre `IS NULL`/`IS NOT NULL` para verificar valores nulos, nunca `= NULL`, evitando el error clasico que hace que la consulta no devuelva resultados esperados.
3. Se evalua paginacion por cursor en vez de `OFFSET` para listados de alto volumen, aplicando el mismo criterio visto en el capitulo de Paginacion del libro de NestJS.
4. Se combinan multiples condiciones con `AND`/`OR` con parentesis explicitos cuando la precedencia podria ser ambigua, evitando errores logicos sutiles.
5. Se conecta esto directamente con las queries que TypeORM/Prisma generan internamente (ver libro de NestJS), entendiendo el SQL real detras de las llamadas del ORM.

## Criterio junior

Un junior debe poder escribir un `SELECT` con `WHERE`, `ORDER BY` y `LIMIT` basico.

## Criterio mid

Un mid debe usar `ILIKE`, manejar correctamente `NULL` con `IS NULL`, y decidir entre `OFFSET` y paginacion por cursor segun el volumen de datos.

## Criterio senior

Un senior debe optimizar consultas de filtrado complejas para datasets grandes, coordinando con indices (ver capitulo siguiente) y decidiendo la estrategia de paginacion apropiada para cada caso de uso.

## Errores comunes

- Usar `= NULL` en vez de `IS NULL`, obteniendo resultados vacios sin entender por que la condicion nunca coincide.
- No usar `ILIKE` cuando se necesita busqueda insensible a mayusculas, perdiendo coincidencias validas por diferencia de capitalizacion.
- Usar `OFFSET` para paginar datasets muy grandes sin considerar el impacto de performance en paginas profundas.
- No usar parentesis explicitos al combinar `AND`/`OR`, generando resultados logicamente incorrectos por precedencia de operadores mal entendida.

## Practica

- [ ] Escribe una consulta con `WHERE`, `ORDER BY` y `LIMIT`/`OFFSET` combinados.
- [ ] Compara `LIKE` e `ILIKE` sobre el mismo texto con distinta capitalizacion.
- [ ] Provoca el error clasico de `= NULL` y corrigelo con `IS NULL`.
- [ ] Combina `AND`/`OR` con parentesis explicitos en una condicion compleja.
- [ ] Investiga como implementarias paginacion por cursor para un listado de alto volumen.

## Preguntas de repaso

- Por que `= NULL` nunca devuelve resultados en SQL, y que deberias usar en su lugar?
- Que diferencia hay entre `LIKE` e `ILIKE` en Postgres?
- Por que la paginacion por `OFFSET` degrada en performance para paginas profundas?
- Que riesgo tiene combinar `AND`/`OR` sin parentesis explicitos en condiciones complejas?
