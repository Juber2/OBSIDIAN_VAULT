---
tags:
  - SQLServer
  - Transacciones
  - SQLServer-Transacciones
Creado: 2026-07-29
Relacionado:
  - Transacciones y Concurrencia
  - SQL SERVER
---

# 01 - ACID en SQL Server

## Que vas a aprender

En este capitulo vas a aplicar los cuatro principios **ACID** (ya estudiados en profundidad en el libro de Postgres) especificamente a la implementacion de SQL Server: sintaxis de `BEGIN TRANSACTION`/`COMMIT`/`ROLLBACK`, y el rol del Transaction Log (ya visto en el capitulo de Arquitectura) en la Durabilidad.

## Explicacion

ACID es teoria de bases de datos relacional **universal**, identica en principio entre Postgres y SQL Server: Atomicidad, Consistencia, Aislamiento, y Durabilidad garantizan lo mismo en ambos motores. La diferencia es puramente sintactica: T-SQL usa `BEGIN TRANSACTION` (o `BEGIN TRAN`) en vez de `BEGIN`, y el resto (`COMMIT`, `ROLLBACK`) es identico. La Durabilidad en SQL Server se garantiza mediante el **Transaction Log** (`.ldf`, ya visto en el capitulo de Fundamentos), el equivalente exacto del WAL de Postgres: cada cambio se escribe primero al log antes de aplicarse a los datos.

Una particularidad util de T-SQL: soporta **savepoints** (`SAVE TRANSACTION nombre`) para poder revertir parcialmente una transaccion hasta un punto intermedio especifico sin deshacer toda la transaccion completa, util en logica transaccional compleja con multiples pasos donde solo una parte necesita revertirse ante cierto error.

## Modelo mental

El modelo mental es identico al ya establecido en Postgres: una transaccion es una promesa formal que se cumple completa o se anula completamente. Los savepoints son como puntos de guardado intermedios dentro de esa misma promesa larga: podes volver a un punto de guardado especifico sin tener que empezar la promesa entera desde cero.

## Ejemplo base

```sql
BEGIN TRANSACTION;

UPDATE Cuentas SET Saldo = Saldo - 100 WHERE Id = 1;
SAVE TRANSACTION DespuesDeDebito;

UPDATE Cuentas SET Saldo = Saldo + 100 WHERE Id = 2;

IF @@ERROR <> 0
BEGIN
    ROLLBACK TRANSACTION DespuesDeDebito;   -- revierte solo hasta el savepoint
END

COMMIT TRANSACTION;
```

## Como se usa en un proyecto real

1. Se envuelve en `BEGIN TRANSACTION`/`COMMIT` cualquier operacion que modifique mas de una tabla, aplicando el mismo criterio ya establecido en Postgres.
2. Se usan savepoints (`SAVE TRANSACTION`) en logica transaccional compleja de multiples pasos donde solo cierta parte necesita poder revertirse selectivamente.
3. Se confia en constraints (ver capitulo de Modelado) como garantia de Consistencia, identico al criterio ya aplicado en Postgres.
4. Se conecta esto con TransactionScope de .NET (ver capitulo de SQL Server con .NET), donde Entity Framework Core envuelve estas mismas operaciones desde C#.
5. Se aprovecha que este conocimiento es completamente transferible desde Postgres, cambiando solo la sintaxis especifica de T-SQL.

## Criterio junior

Un junior debe poder envolver una operacion multi-tabla en `BEGIN TRANSACTION`/`COMMIT`/`ROLLBACK` en T-SQL.

## Criterio mid

Un mid debe usar savepoints para revertir selectivamente parte de una transaccion compleja.

## Criterio senior

Un senior debe diseñar la estrategia de transacciones de un sistema .NET completo, coordinando con TransactionScope de Entity Framework Core.

## Errores comunes

- Modificar varias tablas relacionadas sin una transaccion explicita, el mismo riesgo ya identificado en Postgres.
- No aprovechar savepoints cuando una logica transaccional compleja se beneficiaria de reversion parcial.
- No transferir el conocimiento ya solido de ACID de Postgres, tratando este tema como completamente nuevo.
- No revisar sistematicamente que operaciones multi-tabla del codigo .NET estan realmente envueltas en una transaccion.

## Practica

- [ ] Envuelve una operacion multi-tabla en una transaccion T-SQL y simula un fallo, verificando el rollback.
- [ ] Usa un savepoint para revertir solo parte de una transaccion compleja.
- [ ] Compara la sintaxis de transacciones de T-SQL con la ya dominada en Postgres.
- [ ] Investiga como TransactionScope de .NET envuelve estas mismas operaciones desde C#.
- [ ] Identifica una operacion de StaffTrainingPlatform que deberia estar envuelta en una transaccion.

## Preguntas de repaso

- Que garantiza cada uno de los cuatro principios ACID, identico entre Postgres y SQL Server?
- Que diferencia de sintaxis hay entre iniciar una transaccion en T-SQL versus Postgres?
- Para que sirve un savepoint dentro de una transaccion?
- Como se conecta el Transaction Log de SQL Server con la garantia de Durabilidad?
