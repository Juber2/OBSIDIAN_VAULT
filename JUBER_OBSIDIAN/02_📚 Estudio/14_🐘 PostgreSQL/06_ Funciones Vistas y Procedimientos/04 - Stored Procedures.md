---
tags:
  - PostgreSQL
  - Funciones
  - Postgres-Funciones
Creado: 2026-07-29
Relacionado:
  - Funciones Vistas y Procedimientos
  - POSTGRESQL
---

# 04 - Stored Procedures

## Que vas a aprender

En este capitulo vas a aprender **procedimientos almacenados** (`CREATE PROCEDURE`, distintos de funciones desde Postgres 11), y la diferencia clave: los procedimientos pueden manejar sus propias transacciones internamente (`COMMIT`/`ROLLBACK`), algo que las funciones no pueden hacer.

## Explicacion

Antes de Postgres 11, todo era una "funcion", incluso logica que conceptualmente era mas un procedimiento. Un **procedimiento** (`CREATE PROCEDURE`, invocado con `CALL`) puede contener sus propios `COMMIT`/`ROLLBACK` internos, permitiendo dividir una operacion larga en multiples transacciones mas pequeñas dentro de un unico procedimiento (util para procesamiento batch de grandes volumenes, donde mantener todo en una unica transaccion gigante seria problematico para locks y memoria). Una **funcion** siempre corre dentro de la transaccion que la invoca, sin poder controlar el commit/rollback por su cuenta.

En la practica moderna con NestJS/TypeORM, la mayoria de la logica transaccional se maneja del lado de la aplicacion (ver capitulo de Transacciones del libro de NestJS), y los procedimientos almacenados se reservan para casos especificos de procesamiento masivo de datos directamente en la base, donde el control granular de transacciones dentro del procedimiento aporta un beneficio real.

## Modelo mental

Pensa una funcion como una unica instruccion dentro de una carta ya sellada (la transaccion que la invoco): no puede decidir por su cuenta enviar la carta antes de tiempo. Un procedimiento es como tener autoridad para sellar y enviar varias cartas independientes durante su propia ejecucion, sin depender de una unica carta externa que lo englobe todo.

## Ejemplo base

```sql
CREATE PROCEDURE procesar_lote_pedidos(lote_size integer)
LANGUAGE plpgsql AS $$
DECLARE
    procesados integer := 0;
BEGIN
    LOOP
        UPDATE pedidos SET estado = 'procesado'
        WHERE estado = 'pendiente' AND id IN (
            SELECT id FROM pedidos WHERE estado = 'pendiente' LIMIT lote_size
        );

        GET DIAGNOSTICS procesados = ROW_COUNT;
        EXIT WHEN procesados = 0;

        COMMIT;   -- confirma este lote antes de continuar con el siguiente
    END LOOP;
END;
$$;

CALL procesar_lote_pedidos(1000);
```

## Como se usa en un proyecto real

1. Se usan procedimientos almacenados para procesamiento batch masivo directamente en la base de datos, donde dividir en multiples commits internos evita mantener locks y memoria de una transaccion gigante durante demasiado tiempo.
2. Se prefieren funciones simples sobre procedimientos para la gran mayoria de logica reutilizable que no necesita control transaccional propio.
3. Se documenta claramente cuando un procedimiento hace multiples commits internos, dado que eso significa que un fallo a mitad de camino deja parte del trabajo ya confirmado (sin posibilidad de rollback completo), un comportamiento distinto al de una transaccion atomica tradicional.
4. Se evalua si una tarea de procesamiento masivo realmente necesita vivir como procedimiento de base de datos o si seria mas mantenible como un job de background en NestJS (ver capitulo de BullMQ en ese libro).
5. Se conecta esto con el capitulo de Transacciones del libro de NestJS, entendiendo la diferencia entre control transaccional a nivel de aplicacion y a nivel de procedimiento almacenado.

## Criterio junior

Un junior debe entender la diferencia basica entre una funcion (no puede hacer commit propio) y un procedimiento (si puede).

## Criterio mid

Un mid debe escribir un procedimiento simple con procesamiento en lotes con commits intermedios para una tarea de volumen alto.

## Criterio senior

Un senior debe decidir cuando el procesamiento batch realmente necesita vivir como procedimiento de base de datos versus como job de background en la aplicacion, considerando mantenibilidad, testabilidad y el comportamiento de fallo parcial.

## Errores comunes

- Confundir funciones con procedimientos, intentando hacer `COMMIT` dentro de una funcion (lo cual no esta permitido) y obtener un error.
- No documentar que un procedimiento hace multiples commits internos, sorprendiendo a alguien que asumia atomicidad completa ante un fallo a mitad de camino.
- Usar procedimientos para logica que seria mas mantenible como un job de background en NestJS, perdiendo las ventajas de testing y observabilidad del ecosistema de aplicacion.
- No considerar el comportamiento de fallo parcial (trabajo ya confirmado antes del error) al diseñar un procedimiento con multiples commits.

## Practica

- [ ] Escribe un procedimiento simple con un loop y commits intermedios para procesar datos en lotes.
- [ ] Ejecuta ese procedimiento con `CALL` y verifica el procesamiento por lotes.
- [ ] Explica que pasaria si el procedimiento falla a mitad de su ejecucion, dado los commits intermedios ya realizados.
- [ ] Compara este enfoque con implementar el mismo procesamiento batch como un job de BullMQ en NestJS.
- [ ] Documenta cuando elegirias un procedimiento almacenado sobre un job de aplicacion para procesamiento masivo.

## Preguntas de repaso

- Que puede hacer un procedimiento que una funcion no puede?
- Que riesgo de comportamiento parcial introduce un procedimiento con multiples commits internos?
- Cuando se justifica usar un procedimiento almacenado para procesamiento batch en vez de un job de background en la aplicacion?
- Como te asegurarias de documentar el comportamiento transaccional no obvio de un procedimiento con commits internos?
