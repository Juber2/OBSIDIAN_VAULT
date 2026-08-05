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

# 02 - INSERT UPDATE DELETE

## Que vas a aprender

En este capitulo vas a dominar `INSERT`, `UPDATE`, `DELETE` en T-SQL, y la clausula **`OUTPUT`**, el equivalente exacto de `RETURNING` de Postgres (ya visto en ese libro) que devuelve datos de las filas afectadas sin una consulta adicional.

## Explicacion

`INSERT INTO tabla (columnas) VALUES (...)` agrega filas, con la misma capacidad de insertar multiples filas en una sentencia. `UPDATE`/`DELETE` con `WHERE` siguen exactamente el mismo riesgo critico ya explicado en el libro de Postgres: **omitir el `WHERE` afecta todas las filas**. La clausula `OUTPUT` (sintacticamente distinta de `RETURNING` pero con el mismo proposito) permite capturar valores de las filas insertadas/actualizadas/eliminadas, usando las tablas virtuales `INSERTED` y `DELETED` que T-SQL provee automaticamente dentro de esa clausula.

Una particularidad de T-SQL es que `INSERTED` y `DELETED` tambien estan disponibles dentro de **triggers** (ver capitulo de Funciones y Triggers), un concepto directamente equivalente a `NEW`/`OLD` de PL/pgSQL, pero con nombres y sintaxis distintos.

## Modelo mental

Pensa `OUTPUT` como el mismo recibo inmediato ya explicado con `RETURNING` en Postgres, pero con una sintaxis distinta: en vez de simplemente listar columnas al final de la sentencia, T-SQL te da acceso a dos tablas virtuales especiales (`INSERTED` para el estado nuevo, `DELETED` para el estado anterior) de donde elegis exactamente que datos capturar.

## Ejemplo base

```sql
INSERT INTO Usuarios (Email, Nombre)
OUTPUT INSERTED.Id, INSERTED.CreadoEn
VALUES ('juber@ejemplo.com', 'Juber');

UPDATE Usuarios SET Activo = 0
OUTPUT DELETED.Activo AS ActivoAnterior, INSERTED.Activo AS ActivoNuevo
WHERE Id = 5;

DELETE FROM Usuarios
OUTPUT DELETED.Id, DELETED.Email
WHERE Activo = 0 AND CreadoEn < DATEADD(YEAR, -1, GETDATE());
```

## Como se usa en un proyecto real

1. Se verifica siempre la clausula `WHERE` con un `SELECT` equivalente antes de ejecutar `UPDATE`/`DELETE` en produccion, aplicando el mismo principio de precaucion ya visto en el libro de Postgres.
2. Se aprovecha `OUTPUT` para obtener el ID generado de un `INSERT` sin necesitar una consulta adicional, patron directo hacia Entity Framework Core.
3. Se usa `OUTPUT` con `INSERTED`/`DELETED` para capturar el estado antes y despues en operaciones de auditoria.
4. Se envuelve cualquier `UPDATE`/`DELETE` masivo en una transaccion explicita, aplicando el mismo principio ya visto en Postgres.
5. Se conecta esto con `RETURNING` de Postgres, entendiendo que es la misma necesidad resuelta con sintaxis distinta en cada motor.

## Criterio junior

Un junior debe poder escribir `INSERT`, `UPDATE` y `DELETE` basicos en T-SQL, entendiendo el riesgo critico de omitir `WHERE`.

## Criterio mid

Un mid debe usar `OUTPUT` con `INSERTED`/`DELETED` para evitar consultas adicionales innecesarias.

## Criterio senior

Un senior debe establecer como practica de equipo la verificacion previa obligatoria de cualquier `UPDATE`/`DELETE` sin filtro estrecho en produccion.

## Errores comunes

- Ejecutar un `UPDATE`/`DELETE` sin `WHERE` por error, el mismo riesgo critico ya visto en Postgres.
- No aprovechar `OUTPUT`, haciendo una consulta `SELECT` adicional innecesaria despues de un `INSERT`.
- Confundir `INSERTED` y `DELETED` dentro de un `UPDATE` (donde ambas tablas estan disponibles simultaneamente, representando el estado anterior y nuevo).
- No envolver operaciones de modificacion masiva en una transaccion explicita.

## Practica

- [ ] Inserta multiples filas y usa `OUTPUT` para obtener sus IDs generados.
- [ ] Escribe un `UPDATE` con `OUTPUT` capturando el valor anterior y nuevo de una columna.
- [ ] Verifica con un `SELECT` equivalente antes de ejecutar un `DELETE` con condicion compleja.
- [ ] Compara la sintaxis de `OUTPUT` con `RETURNING` de Postgres para el mismo caso de uso.
- [ ] Documenta una checklist personal antes de cualquier `UPDATE`/`DELETE` en produccion.

## Preguntas de repaso

- Que pasa si ejecutas un `UPDATE` sin clausula `WHERE` en T-SQL?
- Como se relaciona `OUTPUT` de T-SQL con `RETURNING` de Postgres?
- Que diferencia hay entre `INSERTED` y `DELETED` dentro de la clausula `OUTPUT` de un `UPDATE`?
- Que verificacion harias siempre antes de ejecutar un `DELETE` con una condicion compleja en produccion?
