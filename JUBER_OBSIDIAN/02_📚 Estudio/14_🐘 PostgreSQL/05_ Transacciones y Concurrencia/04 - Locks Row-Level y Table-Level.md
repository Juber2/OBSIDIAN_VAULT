---
tags:
  - PostgreSQL
  - Transacciones
  - Postgres-Transacciones
Creado: 2026-07-29
Relacionado:
  - Transacciones y Concurrencia
  - POSTGRESQL
---

# 04 - Locks Row-Level y Table-Level

## Que vas a aprender

En este capitulo vas a aprender los mecanismos explicitos de **locking** de Postgres: `SELECT FOR UPDATE`/`FOR SHARE` a nivel de fila, y locks a nivel de tabla, complementando el aislamiento automatico de MVCC ya visto.

## Explicacion

Aunque MVCC evita que lectores bloqueen escritores, hay casos donde se necesita **bloquear explicitamente** filas para evitar que dos transacciones concurrentes las modifiquen de forma conflictiva: `SELECT ... FOR UPDATE` bloquea las filas seleccionadas contra modificaciones de otras transacciones hasta que la transaccion actual termine, el equivalente exacto del "lock pesimista" visto conceptualmente en el capitulo de Transacciones del libro de NestJS. `SELECT ... FOR SHARE` es mas permisivo: bloquea contra modificaciones pero permite que otras transacciones tambien lean (y hagan su propio `FOR SHARE`) sobre las mismas filas.

Los locks a **nivel de tabla** (`LOCK TABLE`) son mas raros en uso cotidiano, reservados para operaciones administrativas que necesitan exclusividad total sobre una tabla (como ciertas migraciones estructurales). Un riesgo real de cualquier locking explicito es el **deadlock**: dos transacciones bloqueadas mutuamente esperando locks que la otra tiene, que Postgres detecta automaticamente y resuelve abortando una de las dos transacciones con un error que la aplicacion debe manejar.

## Modelo mental

Pensa `SELECT FOR UPDATE` como reservar fisicamente un objeto en una tienda antes de pagarlo: nadie mas puede llevarselo mientras lo tenes reservado, aunque otros clientes puedan seguir mirandolo en la vitrina. Un deadlock es como dos personas que se bloquean mutuamente el paso en un pasillo angosto, cada una esperando que la otra se mueva primero, sin que ninguna pueda avanzar: alguien (Postgres) tiene que intervenir y pedirle a una de las dos que retroceda.

## Ejemplo base

```sql
BEGIN;
SELECT saldo FROM cuentas WHERE id = 1 FOR UPDATE;   -- bloquea esta fila
-- otra transaccion que intente UPDATE o FOR UPDATE sobre id=1 esperara aca
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
COMMIT;   -- libera el lock
```

```sql
-- Detectar deadlocks en los logs (configuracion de postgresql.conf)
-- log_lock_waits = on

SELECT pid, mode, relation::regclass, granted FROM pg_locks WHERE NOT granted;
-- muestra locks que estan esperando ser otorgados, util para diagnosticar contencion
```

## Como se usa en un proyecto real

1. Se usa `SELECT FOR UPDATE` para operaciones donde se lee un valor con intencion de modificarlo inmediatamente despues, evitando condiciones de carrera entre transacciones concurrentes (como descontar stock).
2. Se mantiene el orden de adquisicion de locks consistente entre distintas partes del codigo que bloquean multiples filas, evitando deadlocks causados por orden inverso de bloqueo.
3. Se implementa manejo de errores para deadlocks (reintentar la transaccion completa), dado que Postgres los detecta y aborta una de las transacciones involucradas automaticamente.
4. Se evita mantener transacciones con locks abiertos durante operaciones lentas (como llamadas HTTP externas), minimizando el tiempo que un lock retiene recursos.
5. Se conecta esto directamente con el lock pesimista ya visto conceptualmente en el capitulo de Transacciones del libro de NestJS, ahora con la sintaxis SQL real detras de ese concepto.

## Criterio junior

Un junior debe poder usar `SELECT ... FOR UPDATE` basico para proteger una operacion de lectura-luego-escritura.

## Criterio mid

Un mid debe diagnosticar contencion de locks con `pg_locks`, y manejar errores de deadlock con logica de reintento.

## Criterio senior

Un senior debe diseñar el orden de adquisicion de locks en operaciones complejas que bloquean multiples recursos, previniendo deadlocks por diseño, y establecer convenciones de equipo sobre cuando usar locking explicito.

## Errores comunes

- No usar `FOR UPDATE` en una secuencia leer-luego-modificar, permitiendo condiciones de carrera entre transacciones concurrentes.
- Mantener una transaccion con locks abiertos durante una operacion lenta externa (como una llamada HTTP), reteniendo recursos innecesariamente y aumentando el riesgo de contencion.
- No manejar el error de deadlock en la aplicacion, dejando que se propague como un fallo no manejado en vez de reintentar la operacion.
- Bloquear recursos en orden inconsistente entre distintas partes del codigo, generando deadlocks intermitentes dificiles de reproducir.

## Practica

- [ ] Usa `SELECT ... FOR UPDATE` en una operacion de lectura-luego-escritura de practica.
- [ ] Simula dos transacciones concurrentes bloqueadas en la misma fila y observa el comportamiento de espera.
- [ ] Provoca un deadlock intencional entre dos transacciones y observa como Postgres lo resuelve.
- [ ] Implementa logica de reintento simple para manejar un deadlock detectado.
- [ ] Revisa `pg_locks` durante una situacion de contencion simulada.

## Preguntas de repaso

- Que diferencia hay entre `SELECT FOR UPDATE` y `SELECT FOR SHARE`?
- Como detecta y resuelve Postgres un deadlock automaticamente?
- Por que mantener una transaccion abierta durante una llamada externa lenta es riesgoso?
- Que practica de diseño previene deadlocks causados por orden inconsistente de bloqueo?
