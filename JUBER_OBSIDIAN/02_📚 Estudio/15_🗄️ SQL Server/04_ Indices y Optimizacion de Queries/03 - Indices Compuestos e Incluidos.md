---
tags:
  - SQLServer
  - Performance
  - SQLServer-Indices
Creado: 2026-07-29
Relacionado:
  - Indices y Optimizacion de Queries
  - SQL SERVER
---

# 03 - Indices Compuestos e Incluidos

## Que vas a aprender

En este capitulo vas a aprender **indices compuestos** (ya conocido el concepto general de Postgres) y **columnas incluidas** (`INCLUDE`), una feature especifica de SQL Server sin equivalente directo en Postgres, que permite crear **indices de cobertura** (covering indexes) muy eficientes.

## Explicacion

Un indice compuesto en T-SQL sigue exactamente la misma regla de orden de columnas ya estudiada en Postgres: la columna de igualdad exacta primero, la de rango/orden despues. Lo especifico de SQL Server es la clausula **`INCLUDE`**: permite agregar columnas adicionales al indice **sin que formen parte de la clave de busqueda**, solo para que sus valores esten disponibles directamente en el indice, evitando que SQL Server necesite ir a buscar esas columnas en la tabla real (un **key lookup**, operacion adicional costosa que aparece en el Execution Plan cuando el indice no cubre todas las columnas necesitadas).

Un indice que contiene **todas** las columnas que una consulta necesita (tanto en la clave como en `INCLUDE`) se llama **covering index**, permitiendo que SQL Server resuelva la consulta completa leyendo unicamente el indice, sin tocar la tabla real en absoluto, siendo la optimizacion mas efectiva posible para una consulta especifica de alta frecuencia.

## Modelo mental

Pensa un indice compuesto normal como un catalogo de biblioteca ordenado por autor y titulo: eficiente para buscar, pero si tambien queres saber el numero de paginas de cada libro, tenes que ir fisicamente al estante a buscarlo (key lookup). Un indice con `INCLUDE` es como si ese mismo catalogo tuviera ademas anotado el numero de paginas directamente en la ficha, sin necesitar ir al estante para nada: todo lo que necesitas esta en el catalogo mismo.

## Ejemplo base

```sql
-- Indice compuesto, mismo criterio de orden ya visto en Postgres
CREATE NONCLUSTERED INDEX IX_Pedidos_Cliente_Fecha
ON Pedidos (ClienteId, CreadoEn DESC);

-- Covering index: incluye Total y Estado sin que formen parte de la clave de busqueda
CREATE NONCLUSTERED INDEX IX_Pedidos_Cliente_Fecha_Covering
ON Pedidos (ClienteId, CreadoEn DESC)
INCLUDE (Total, Estado);

-- Esta consulta se resuelve completamente desde el indice, sin tocar la tabla (covering)
SELECT ClienteId, CreadoEn, Total, Estado
FROM Pedidos
WHERE ClienteId = 42
ORDER BY CreadoEn DESC;
```

## Como se usa en un proyecto real

1. Se diseñan indices compuestos con el mismo criterio de orden de columnas ya dominado en Postgres (igualdad primero, rango/orden despues).
2. Se agregan columnas frecuentemente consultadas (pero no usadas para filtrar) con `INCLUDE`, creando covering indexes para consultas de alta frecuencia.
3. Se identifica la necesidad de un covering index revisando el Execution Plan y buscando operaciones de **Key Lookup**, que revelan que el indice actual no cubre completamente la consulta.
4. Se balancea el numero y tamaño de indices con `INCLUDE` contra el costo adicional de mantenimiento en cada escritura, evitando incluir columnas innecesarias "por si acaso".
5. Se documenta el proposito de covering indexes no obvios, dado que su diseño esta atado a un patron de consulta especifico.

## Criterio junior

Un junior debe poder crear un indice compuesto basico aplicando el criterio de orden ya conocido de Postgres.

## Criterio mid

Un mid debe usar `INCLUDE` para crear covering indexes que eliminen operaciones de Key Lookup en consultas frecuentes.

## Criterio senior

Un senior debe diseñar la estrategia de covering indexes para las consultas mas criticas de un sistema, balanceando el beneficio de lectura contra el costo de mantenimiento de escritura.

## Errores comunes

- No usar `INCLUDE` cuando una consulta frecuente sufre de Key Lookups repetidos, perdiendo una optimizacion significativa disponible.
- Incluir demasiadas columnas innecesarias en `INCLUDE`, aumentando el tamaño y costo de mantenimiento del indice sin beneficio real.
- No revisar el Execution Plan buscando operaciones de Key Lookup como señal de necesitar un covering index.
- Confundir columnas de la clave del indice con columnas `INCLUDE`, sin entender la diferencia de proposito entre ambas.

## Practica

- [ ] Crea un indice compuesto simple y observa una operacion de Key Lookup en el Execution Plan de una consulta que necesita columnas adicionales.
- [ ] Convierte ese indice en un covering index agregando `INCLUDE` con las columnas necesarias.
- [ ] Verifica en el Execution Plan que la operacion de Key Lookup desaparecio.
- [ ] Compara el tamaño del indice antes y despues de agregar columnas `INCLUDE`.
- [ ] Diseña un covering index para una consulta real y frecuente de tu proyecto.

## Preguntas de repaso

- Que es un Key Lookup y que revela sobre un indice existente?
- Que diferencia hay entre una columna en la clave de un indice y una columna `INCLUDE`?
- Que es un covering index y por que es la optimizacion mas efectiva para una consulta especifica?
- Que tradeoff hay que considerar antes de agregar muchas columnas con `INCLUDE`?
