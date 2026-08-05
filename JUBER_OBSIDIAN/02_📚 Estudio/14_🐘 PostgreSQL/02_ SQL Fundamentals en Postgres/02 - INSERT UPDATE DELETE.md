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

# 02 - INSERT UPDATE DELETE

## Que vas a aprender

En este capitulo vas a dominar las operaciones de modificacion de datos: `INSERT`, `UPDATE`, `DELETE`, y la clausula `RETURNING` de Postgres (una feature que no todos los motores tienen) que devuelve datos de las filas afectadas sin necesitar una consulta adicional.

## Explicacion

`INSERT INTO tabla (columnas) VALUES (...)` agrega filas nuevas; se puede insertar multiples filas en una sola sentencia separando grupos de valores con comas, mas eficiente que multiples `INSERT` individuales. `UPDATE tabla SET columna = valor WHERE condicion` modifica filas existentes: **omitir el `WHERE` actualiza todas las filas de la tabla**, uno de los errores mas peligrosos y comunes en SQL. `DELETE FROM tabla WHERE condicion` elimina filas, con el mismo riesgo si se olvida el `WHERE`.

`RETURNING` es una extension de Postgres (no estandar SQL puro, aunque cada vez mas motores la adoptan) que devuelve columnas de las filas afectadas por un `INSERT`, `UPDATE` o `DELETE`, evitando tener que hacer una consulta `SELECT` separada para obtener, por ejemplo, el ID generado automaticamente de una fila recien insertada.

## Modelo mental

Pensa `WHERE` como el filtro de precision de un francotirador: sin el, cualquier operacion de modificacion (`UPDATE`/`DELETE`) afecta a **todo el batallon** (todas las filas de la tabla), no solo al objetivo especifico que se pretendia. `RETURNING` es como pedir un recibo inmediato de la operacion que acabas de hacer, sin necesitar ir a preguntar por separado que fue exactamente lo que paso.

## Ejemplo base

```sql
INSERT INTO usuarios (email, nombre) VALUES ('juber@ejemplo.com', 'Juber')
RETURNING id, creado_en;

INSERT INTO productos (nombre, precio) VALUES
    ('Producto A', 100),
    ('Producto B', 200);

UPDATE usuarios SET activo = false WHERE id = 5;
-- SIN WHERE, esto desactivaria TODOS los usuarios

DELETE FROM usuarios WHERE activo = false AND creado_en < now() - interval '1 year';

UPDATE productos SET precio = precio * 1.1 WHERE precio < 50 RETURNING id, precio;
```

## Como se usa en un proyecto real

1. Se verifica siempre la clausula `WHERE` con una consulta `SELECT` equivalente **antes** de ejecutar un `UPDATE`/`DELETE` en produccion, confirmando exactamente que filas serian afectadas.
2. Se aprovecha `RETURNING` para obtener el ID generado de un `INSERT` sin necesitar una consulta adicional, patron comun al integrar con ORMs como TypeORM.
3. Se usa `INSERT` con multiples filas en una sola sentencia para cargas de datos masivas, mucho mas eficiente que insertar fila por fila.
4. Se envuelve cualquier `UPDATE`/`DELETE` masivo en una transaccion (ver capitulo de Transacciones) que permita revertir si algo sale mal antes de confirmar.
5. Se documenta cualquier operacion de modificacion masiva ejecutada manualmente en produccion, como parte de un registro de cambios operacionales.

## Criterio junior

Un junior debe poder escribir `INSERT`, `UPDATE` y `DELETE` basicos, entendiendo el riesgo critico de omitir `WHERE`.

## Criterio mid

Un mid debe usar `RETURNING` para evitar consultas adicionales innecesarias, y verificar el alcance de un `UPDATE`/`DELETE` antes de ejecutarlo en produccion.

## Criterio senior

Un senior debe establecer como practica de equipo la verificacion previa obligatoria de cualquier `UPDATE`/`DELETE` sin filtro estrecho en produccion, y definir el proceso de aprobacion para operaciones de modificacion masiva.

## Errores comunes

- Ejecutar un `UPDATE` o `DELETE` sin `WHERE` por error, afectando todas las filas de una tabla de forma irreversible sin una transaccion de respaldo.
- No verificar con un `SELECT` equivalente antes de un `UPDATE`/`DELETE` con condiciones complejas, confiando en que el filtro es correcto sin confirmarlo.
- No aprovechar `RETURNING`, haciendo una consulta `SELECT` adicional innecesaria despues de un `INSERT` para obtener el ID generado.
- No envolver operaciones de modificacion masiva en una transaccion, perdiendo la posibilidad de revertir si algo sale mal a mitad de camino.

## Practica

- [ ] Inserta multiples filas en una unica sentencia `INSERT` y usa `RETURNING` para obtener sus IDs.
- [ ] Escribe un `UPDATE` con `WHERE` y verifica antes con un `SELECT` equivalente que filas serian afectadas.
- [ ] Simula (en una tabla de practica) el error de omitir `WHERE` en un `UPDATE` y observa el impacto.
- [ ] Envuelve una operacion de modificacion masiva en una transaccion que puedas revertir con `ROLLBACK`.
- [ ] Documenta una checklist personal de verificacion antes de ejecutar cualquier `UPDATE`/`DELETE` en produccion.

## Preguntas de repaso

- Que pasa si ejecutas un `UPDATE` sin clausula `WHERE`?
- Para que sirve `RETURNING` y que consulta adicional evita?
- Que verificacion harias siempre antes de ejecutar un `DELETE` con una condicion compleja en produccion?
- Por que envolver una operacion masiva en una transaccion da una red de seguridad adicional?
