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

# 01 - ACID en Postgres

## Que vas a aprender

En este capitulo vas a repasar los cuatro principios **ACID** (Atomicidad, Consistencia, Aislamiento, Durabilidad) aplicados especificamente a como Postgres los implementa, sentando la base para los temas de concurrencia y MVCC de este capitulo.

## Explicacion

**Atomicidad** garantiza que una transaccion se aplica completa o no se aplica en absoluto (`BEGIN`/`COMMIT`/`ROLLBACK`, ya visto conceptualmente en el libro de NestJS): si algo falla a mitad de camino, todo se revierte. **Consistencia** garantiza que una transaccion lleva la base de datos de un estado valido a otro estado valido, respetando todos los constraints (ver capitulo de Modelado). **Aislamiento** define que tan visibles son los cambios de una transaccion no confirmada para otras transacciones concurrentes (profundizado en el tema de niveles de aislamiento). **Durabilidad** garantiza que, una vez confirmada (`COMMIT`), una transaccion sobrevive incluso a una caida del servidor inmediatamente despues, gracias al **WAL** (Write-Ahead Log, ya mencionado en el capitulo de Administracion), que registra los cambios en disco antes de confirmar.

## Modelo mental

Pensa una transaccion ACID como una promesa formal notarizada: Atomicidad es que la promesa se cumple completa o se anula completamente, nunca a medias. Consistencia es que la promesa nunca deja las cosas en un estado que viole las reglas acordadas. Aislamiento es que dos promesas que se estan negociando simultaneamente no se interfieren de forma incorrecta entre si. Durabilidad es que, una vez firmada la promesa, queda registrada de forma permanente aunque se corte la luz un segundo despues.

## Ejemplo base

```sql
BEGIN;

UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;

-- Si ambas operaciones tienen exito:
COMMIT;

-- Si algo falla (por ejemplo, un constraint de saldo no negativo):
-- ROLLBACK;   -- ninguna de las dos actualizaciones queda aplicada
```

## Como se usa en un proyecto real

1. Se envuelve en una transaccion cualquier operacion que modifique mas de una tabla y donde un fallo a mitad de camino dejaria datos inconsistentes, aplicando el mismo principio ya visto en el capitulo de Transacciones del libro de NestJS.
2. Se confia en constraints (ver capitulo de Modelado) como garantia de consistencia, incluso si la logica de aplicacion tambien valida lo mismo.
3. Se entiende que Durabilidad tiene un costo de performance (escribir al WAL antes de confirmar), relevante al configurar `synchronous_commit` para casos donde se acepta un pequeño riesgo a cambio de mayor velocidad.
4. Se conecta ACID con MVCC (tema siguiente): MVCC es el mecanismo especifico que Postgres usa para implementar Aislamiento sin bloquear lectores contra escritores.
5. Se documenta claramente en el codigo cuando una operacion multi-tabla necesita una transaccion explicita, evitando dejarlo como una decision implicita no revisada.

## Criterio junior

Un junior debe poder envolver una operacion multi-tabla en `BEGIN`/`COMMIT`/`ROLLBACK` y explicar que garantiza cada letra de ACID.

## Criterio mid

Un mid debe identificar que operaciones de su aplicacion necesitan una transaccion explicita, y entender el rol del WAL en la durabilidad.

## Criterio senior

Un senior debe diseñar la estrategia de transacciones de un sistema completo, balanceando durabilidad estricta contra performance cuando corresponda, y documentando esas decisiones.

## Errores comunes

- Modificar varias tablas relacionadas sin una transaccion explicita, dejando datos inconsistentes ante un fallo parcial.
- No entender que la Consistencia de ACID depende de constraints bien diseñados, asumiendo que la transaccion por si sola garantiza reglas de negocio.
- Ignorar el costo de performance de la Durabilidad estricta en casos donde un pequeño riesgo controlado seria aceptable a cambio de mayor velocidad.
- No revisar sistematicamente que operaciones multi-tabla del codigo realmente estan envueltas en una transaccion.

## Practica

- [ ] Envuelve una operacion multi-tabla en una transaccion y simula un fallo a mitad de camino, verificando el rollback.
- [ ] Explica con tus palabras que garantiza cada letra de ACID en el contexto de Postgres.
- [ ] Investiga que hace `synchronous_commit` y el tradeoff que representa.
- [ ] Identifica una operacion de tu proyecto real que deberia estar envuelta en una transaccion y verifica si lo esta.
- [ ] Conecta este capitulo con el capitulo de Transacciones ya visto en el libro de NestJS.

## Preguntas de repaso

- Que garantiza especificamente cada uno de los cuatro principios ACID?
- Que rol cumple el WAL en la garantia de Durabilidad?
- Que mecanismo especifico usa Postgres para implementar Aislamiento sin bloquear lectores contra escritores?
- Que tradeoff representa ajustar `synchronous_commit`?
