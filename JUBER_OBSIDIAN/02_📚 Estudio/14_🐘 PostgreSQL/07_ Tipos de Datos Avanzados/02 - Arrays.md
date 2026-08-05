---
tags:
  - PostgreSQL
  - Tipos-de-Datos
  - Postgres-Avanzado
Creado: 2026-07-29
Relacionado:
  - Tipos de Datos Avanzados
  - POSTGRESQL
---

# 02 - Arrays

## Que vas a aprender

En este capitulo vas a aprender a usar el tipo **array** nativo de Postgres, una feature que la mayoria de otros motores relacionales no tienen: como declarar columnas array, consultarlas, e indexarlas, y cuando son apropiadas versus cuando una tabla relacional seria mejor.

## Explicacion

Postgres permite que una columna contenga un **array** de valores de cualquier tipo (`text[]`, `integer[]`), consultable con operadores especificos: `@>` (contiene todos los elementos indicados), `&&` (se superpone, tiene al menos un elemento en comun), y la funcion `unnest()` (expande un array en filas individuales, util para hacer joins o agregaciones sobre elementos de un array).

Al igual que con `jsonb`, los arrays son una herramienta poderosa para casos genuinamente apropiados (una lista corta de tags simples, colores disponibles de un producto) pero **no reemplazan** una relacion uno a muchos real cuando los "elementos" del array tienen su propia identidad, atributos adicionales, o necesitan integridad referencial (en ese caso, una tabla relacional separada sigue siendo el diseño correcto, como ya visto en el capitulo de Relaciones).

## Modelo mental

Pensa una columna array como una casilla de un formulario que permite escribir varias palabras clave separadas por comas, en vez de tener que crear un formulario separado por cada palabra clave. Es comodo para listas simples y cortas, pero si cada "palabra clave" en realidad necesitara su propia ficha con mas informacion (fecha de creacion, quien la agrego, su propia relacion con otras entidades), entonces necesitas una tabla real, no una casilla de texto con comas.

## Ejemplo base

```sql
CREATE TABLE productos (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre text NOT NULL,
    tags text[]
);

INSERT INTO productos (nombre, tags) VALUES
    ('Camisa Azul', ARRAY['ropa', 'verano', 'oferta']);

SELECT * FROM productos WHERE tags @> ARRAY['oferta'];        -- contiene "oferta"
SELECT * FROM productos WHERE tags && ARRAY['oferta', 'invierno'];  -- se superpone con alguno

SELECT nombre, unnest(tags) AS tag FROM productos;   -- expande el array en filas

CREATE INDEX idx_productos_tags ON productos USING gin (tags);
```

## Como se usa en un proyecto real

1. Se usa un array para listas cortas de valores simples sin identidad propia (tags, colores disponibles), evitando el overhead de una tabla intermedia para casos genuinamente simples.
2. Se indexa con GIN cualquier columna array consultada frecuentemente por contencion o superposicion, aplicando el mismo criterio ya visto en el capitulo de Indices.
3. Se prefiere una tabla relacional real (con su propia clave primaria y posibles atributos adicionales) cuando los elementos necesitan mas que un valor simple, aplicando el criterio ya visto en el capitulo de Relaciones NaM.
4. Se usa `unnest()` para convertir un array en filas cuando se necesita agregar o unir sobre elementos individuales del array.
5. Se documenta la decision de usar array versus tabla relacional para un caso especifico, dado que es una decision de diseño con tradeoffs reales.

## Criterio junior

Un junior debe poder crear una columna array, insertar valores, y consultarla con `@>`.

## Criterio mid

Un mid debe usar `unnest()` para trabajar con elementos individuales de un array, e indexar arrays consultados frecuentemente.

## Criterio senior

Un senior debe decidir con criterio cuando un array es apropiado versus cuando una relacion muchos a muchos real seria el diseño correcto, evitando el abuso de arrays para modelar datos que en realidad tienen identidad propia.

## Errores comunes

- Usar un array para modelar una relacion muchos a muchos real, donde los elementos tienen atributos adicionales o necesitan integridad referencial que un array no puede proveer.
- No indexar con GIN una columna array consultada frecuentemente, generando escaneos completos costosos.
- Confundir `@>` (contiene todos) con `&&` (se superpone con alguno), obteniendo resultados inesperados en la consulta.
- No considerar `unnest()` cuando se necesita trabajar con elementos individuales, intentando manipular el array completo de formas mas complicadas.

## Practica

- [ ] Crea una columna array de texto e inserta datos de ejemplo.
- [ ] Consulta con `@>` y `&&`, comparando la diferencia de resultados.
- [ ] Usa `unnest()` para expandir el array en filas y agregar sobre sus elementos.
- [ ] Indexa la columna array con GIN.
- [ ] Identifica un caso de tu proyecto donde un array seria apropiado, y otro donde en realidad necesitarias una tabla relacional.

## Preguntas de repaso

- Que diferencia hay entre los operadores `@>` y `&&` sobre columnas array?
- Para que sirve `unnest()` y cuando lo usarias?
- Cuando un array es el diseño correcto, y cuando en realidad se necesita una tabla relacional?
- Que tipo de indice usarias para una columna array consultada frecuentemente?
