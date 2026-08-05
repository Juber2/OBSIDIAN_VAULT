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

# 04 - JOINs

## Que vas a aprender

En este capitulo vas a dominar los distintos tipos de `JOIN`: `INNER`, `LEFT`, `RIGHT`, `FULL OUTER`, y `CROSS JOIN`, entendiendo exactamente que filas devuelve cada uno, la base tecnica detras de las relaciones ya vistas conceptualmente en TypeORM.

## Explicacion

`INNER JOIN` devuelve solo las filas que tienen coincidencia en **ambas** tablas relacionadas. `LEFT JOIN` (el mas usado en la practica) devuelve **todas** las filas de la tabla izquierda, con las columnas de la tabla derecha en `NULL` cuando no hay coincidencia, util para "todos los usuarios, con sus pedidos si los tienen" sin excluir usuarios sin pedidos. `RIGHT JOIN` es el equivalente invertido (poco usado en la practica, dado que reescribir como `LEFT JOIN` intercambiando el orden de tablas suele ser mas legible). `FULL OUTER JOIN` combina ambos, devolviendo todas las filas de ambas tablas, coincidan o no.

Es critico entender donde poner condiciones adicionales: una condicion en la clausula `ON` del `JOIN` filtra **antes** de unir (afectando que filas se combinan), mientras que la misma condicion en `WHERE` filtra **despues** de unir, lo cual puede convertir efectivamente un `LEFT JOIN` en el equivalente de un `INNER JOIN` si no se entiende esta diferencia, un error sutil muy comun.

## Modelo mental

Pensa `INNER JOIN` como una lista de parejas de baile donde solo aparecen las personas que efectivamente tienen pareja. `LEFT JOIN` es la lista completa de todas las personas del lado izquierdo, con "sin pareja" anotado junto a quien no baila con nadie del otro lado, en vez de excluirlos de la lista completamente.

## Ejemplo base

```sql
-- INNER JOIN: solo usuarios que SI tienen pedidos
SELECT u.nombre, p.total
FROM usuarios u
INNER JOIN pedidos p ON p.usuario_id = u.id;

-- LEFT JOIN: TODOS los usuarios, con NULL en pedidos si no tienen ninguno
SELECT u.nombre, p.total
FROM usuarios u
LEFT JOIN pedidos p ON p.usuario_id = u.id;

-- Trampa comun: esta condicion en WHERE anula el efecto de LEFT JOIN
SELECT u.nombre, p.total
FROM usuarios u
LEFT JOIN pedidos p ON p.usuario_id = u.id
WHERE p.total > 100;   -- excluye usuarios sin pedidos, comportandose como INNER JOIN

-- Correcto: la condicion adicional va en el ON, no en WHERE
SELECT u.nombre, p.total
FROM usuarios u
LEFT JOIN pedidos p ON p.usuario_id = u.id AND p.total > 100;
```

## Como se usa en un proyecto real

1. Se usa `LEFT JOIN` cuando se necesita la entidad "principal" completa aunque la relacion sea opcional (todos los usuarios, tengan o no pedidos).
2. Se ubica cuidadosamente cualquier condicion adicional en `ON` versus `WHERE`, entendiendo el efecto distinto de cada ubicacion sobre un `LEFT JOIN`.
3. Se evita `RIGHT JOIN` en favor de reescribir como `LEFT JOIN` intercambiando tablas, por legibilidad y consistencia del equipo.
4. Se conecta esto directamente con las relaciones de TypeORM vistas en el libro de NestJS, entendiendo el SQL real que genera `relations` o un `QueryBuilder` con joins.
5. Se revisa el plan de ejecucion (ver capitulo de Indices) de joins complejos para verificar que se estan usando indices apropiados en las columnas de union.

## Criterio junior

Un junior debe poder escribir un `INNER JOIN` y un `LEFT JOIN` basico entre dos tablas relacionadas.

## Criterio mid

Un mid debe entender la diferencia entre poner una condicion en `ON` versus `WHERE` en un `LEFT JOIN`, y cuando cada una es la correcta.

## Criterio senior

Un senior debe optimizar joins complejos entre multiples tablas, verificando planes de ejecucion, y conectar el entendimiento de SQL puro con lo que genera un ORM como TypeORM.

## Errores comunes

- Poner una condicion de filtrado adicional en `WHERE` en vez de `ON`, convirtiendo accidentalmente un `LEFT JOIN` en el equivalente de un `INNER JOIN`.
- Usar `RIGHT JOIN` cuando reescribir como `LEFT JOIN` con las tablas intercambiadas seria mas legible y consistente.
- No entender que `INNER JOIN` excluye filas sin coincidencia, usandolo cuando en realidad se necesitaba `LEFT JOIN`.
- No revisar el plan de ejecucion de joins complejos, dejando pasar queries lentas por falta de indices en columnas de union.

## Practica

- [ ] Escribe un `INNER JOIN` y un `LEFT JOIN` entre dos tablas relacionadas y compara los resultados.
- [ ] Provoca el error de condicion en `WHERE` en vez de `ON` con un `LEFT JOIN` y observa el cambio de comportamiento.
- [ ] Corrige ese error moviendo la condicion al lugar correcto.
- [ ] Escribe un `FULL OUTER JOIN` y explica que filas adicionales aparecen respecto a un `LEFT JOIN`.
- [ ] Conecta un `LEFT JOIN` que escribiste con el equivalente `relations` o `QueryBuilder` de TypeORM.

## Preguntas de repaso

- Que diferencia hay entre `INNER JOIN` y `LEFT JOIN` en cuanto a filas sin coincidencia?
- Por que una condicion adicional en `WHERE` puede anular el efecto de un `LEFT JOIN`?
- Cuando usarias `FULL OUTER JOIN` en vez de `LEFT JOIN`?
- Por que se prefiere reescribir un `RIGHT JOIN` como `LEFT JOIN` con tablas intercambiadas?
