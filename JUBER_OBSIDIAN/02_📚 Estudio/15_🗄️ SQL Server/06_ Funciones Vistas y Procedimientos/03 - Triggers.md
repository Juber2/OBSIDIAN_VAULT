---
tags:
  - SQLServer
  - Funciones
  - SQLServer-Funciones
Creado: 2026-07-29
Relacionado:
  - Funciones Vistas y Procedimientos
  - SQL SERVER
---

# 03 - Triggers

## Que vas a aprender

En este capitulo vas a aplicar triggers en T-SQL, identicos en proposito a los ya estudiados en Postgres, con la sintaxis especifica de SQL Server usando las tablas virtuales **`INSERTED`** y **`DELETED`** (ya mencionadas en el capitulo de INSERT/UPDATE/DELETE, el equivalente exacto de `NEW`/`OLD` de PL/pgSQL).

## Explicacion

Un trigger `AFTER INSERT/UPDATE/DELETE` (el tipo mas comun, equivalente a los ya vistos en Postgres) se ejecuta despues de la operacion, con acceso a `INSERTED` (filas nuevas o el estado nuevo en un UPDATE) y `DELETED` (filas eliminadas o el estado anterior en un UPDATE), disponibles como **tablas completas** (no una sola fila a la vez como `NEW`/`OLD` de PL/pgSQL), dado que un trigger en T-SQL se dispara **una vez por sentencia**, no una vez por fila, siendo esta una diferencia de diseño importante frente al comportamiento de Postgres.

Esta diferencia significa que un trigger de T-SQL debe estar preparado para procesar **multiples filas simultaneamente** (por ejemplo, un `UPDATE` que afecta 100 filas dispara el trigger una sola vez, con `INSERTED`/`DELETED` conteniendo las 100 filas), a diferencia de un trigger `FOR EACH ROW` de Postgres que se ejecuta individualmente por cada fila afectada.

## Modelo mental

Pensa un trigger de T-SQL como una alarma que suena **una unica vez** cuando termina una operacion completa, entregandote una lista completa de todo lo que cambio de una vez (`INSERTED`/`DELETED` como tablas), en vez de sonar individualmente por cada elemento que cambio como haria el equivalente `FOR EACH ROW` de Postgres.

## Ejemplo base

```sql
CREATE TRIGGER trg_Pedidos_Actualizado
ON Pedidos
AFTER UPDATE
AS
BEGIN
    UPDATE p
    SET ActualizadoEn = GETDATE()
    FROM Pedidos p
    INNER JOIN INSERTED i ON i.Id = p.Id;
END;

-- Trigger de auditoria: procesa TODAS las filas afectadas por la sentencia, no una por vez
CREATE TRIGGER trg_Auditoria_Pedidos
ON Pedidos
AFTER UPDATE, DELETE
AS
BEGIN
    INSERT INTO AuditoriaPedidos (PedidoId, Accion, DatosAnteriores)
    SELECT Id, 'MODIFICADO', (SELECT * FROM DELETED d WHERE d.Id = Id FOR JSON AUTO)
    FROM DELETED;
END;
```

## Como se usa en un proyecto real

1. Se diseñan triggers de T-SQL pensando explicitamente en **conjuntos de filas**, nunca asumiendo que solo una fila fue afectada, un error comun de quien viene de un modelo `FOR EACH ROW` de Postgres.
2. Se usa `INSERTED`/`DELETED` con JOINs (como en el ejemplo) en vez de loops fila por fila, aprovechando el procesamiento basado en conjuntos que T-SQL favorece.
3. Se evita logica de negocio compleja en triggers, aplicando el mismo criterio ya establecido en Postgres.
4. Se documenta cada trigger existente, dado que su comportamiento no es visible leyendo solo el codigo de la aplicacion.
5. Se testea un trigger especificamente con operaciones que afectan multiples filas simultaneamente, verificando que maneja correctamente ese caso.

## Criterio junior

Un junior debe poder crear un trigger simple usando `INSERTED`/`DELETED`.

## Criterio mid

Un mid debe diseñar triggers que procesan correctamente multiples filas afectadas por una unica sentencia, sin asumir una sola fila.

## Criterio senior

Un senior debe decidir con criterio cuando un trigger es apropiado, y auditar que triggers existentes manejan correctamente el procesamiento basado en conjuntos.

## Errores comunes

- Asumir que un trigger de T-SQL se dispara una vez por fila (como `FOR EACH ROW` de Postgres), escribiendo logica que solo funciona correctamente si una unica fila fue afectada.
- No probar un trigger con operaciones que afectan multiples filas simultaneamente, dejando pasar bugs que solo aparecen con `UPDATE`/`DELETE` masivos.
- Esconder logica de negocio compleja en triggers, el mismo riesgo ya identificado en Postgres.
- No documentar triggers existentes, generando comportamiento "misterioso" para el equipo.

## Practica

- [ ] Crea un trigger `AFTER UPDATE` que actualice una columna de auditoria.
- [ ] Prueba ese trigger con un `UPDATE` que afecte multiples filas simultaneamente, verificando que funciona correctamente para todas.
- [ ] Crea un trigger de auditoria que registre cambios usando `DELETED`.
- [ ] Compara explicitamente este modelo "por sentencia" con el modelo `FOR EACH ROW` ya dominado de Postgres.
- [ ] Documenta los triggers de un esquema de practica.

## Preguntas de repaso

- Que diferencia fundamental hay entre como se dispara un trigger de T-SQL versus un trigger `FOR EACH ROW` de Postgres?
- Por que `INSERTED`/`DELETED` son tablas completas y no una unica fila?
- Que riesgo tiene diseñar un trigger asumiendo que solo una fila fue afectada?
- Como probarias que un trigger maneja correctamente una operacion que afecta multiples filas?
