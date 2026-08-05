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

# 02 - Niveles de Aislamiento

## Que vas a aprender

En este capitulo vas a aprender los niveles de aislamiento de SQL Server: **Read Uncommitted**, **Read Committed** (default), **Repeatable Read**, **Serializable** (los mismos conceptos ya vistos en Postgres, con un nivel adicional), y **Snapshot Isolation**, una alternativa especifica de SQL Server basada en versionado similar a MVCC de Postgres.

## Explicacion

Los primeros cuatro niveles siguen el estandar SQL identico en concepto a lo ya estudiado en Postgres, aunque con una diferencia critica de implementacion: por defecto, SQL Server usa **locking pesimista** para lograr estos niveles de aislamiento (a diferencia de Postgres, que usa MVCC por defecto para Read Committed y superiores, ver capitulo de MVCC de ese libro), lo que significa que en SQL Server, lectores **si pueden bloquear** a escritores bajo el modelo de locking tradicional, un comportamiento distinto al de Postgres que sorprende a quien viene de ese motor.

**Read Uncommitted** (unico entre los cuatro basicos sin equivalente recomendado en Postgres) permite leer datos **no confirmados** de otras transacciones (dirty reads), sacrificando correccion por velocidad, generalmente desaconsejado salvo casos muy especificos. **Snapshot Isolation** es la respuesta de SQL Server al modelo MVCC de Postgres: activandolo explicitamente (`ALTER DATABASE ... SET ALLOW_SNAPSHOT_ISOLATION ON`), las lecturas dejan de bloquear escrituras, comportandose de forma mucho mas parecida al Postgres que ya conoces.

## Modelo mental

Pensa el locking tradicional de SQL Server como un modelo donde, por defecto, alguien leyendo un documento fisicamente le impide a otra persona escribir sobre el mismo documento simultaneamente (un bloqueo real). Snapshot Isolation es activar el modelo "estilo Postgres" donde cada lector recibe su propia fotocopia del documento tal como estaba en su momento de inicio, sin necesitar bloquear al escritor real, exactamente el comportamiento MVCC que ya conoces.

## Ejemplo base

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;   -- default

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Activar Snapshot Isolation a nivel de base de datos (comportamiento similar a Postgres MVCC)
ALTER DATABASE MiApp SET ALLOW_SNAPSHOT_ISOLATION ON;

BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
SELECT * FROM Pedidos WHERE Id = 1;   -- no bloquea ni es bloqueado por escritores concurrentes
COMMIT;
```

## Como se usa en un proyecto real

1. Se usa Read Committed (el default) para la mayoria de operaciones, entendiendo que en SQL Server esto puede implicar bloqueos breves distintos al comportamiento no bloqueante de Postgres por defecto.
2. Se evalua activar Snapshot Isolation en bases de datos donde el comportamiento de bloqueo tradicional causa contencion significativa, acercando el comportamiento al ya conocido de Postgres.
3. Se evita Read Uncommitted salvo un caso muy especifico donde dirty reads sean aceptables (reportes aproximados de baja criticidad).
4. Se conecta esto con el capitulo de Niveles de Aislamiento de Postgres, entendiendo las diferencias de implementacion (locking vs MVCC por defecto) detras de conceptos con nombres similares.
5. Se documenta la eleccion de nivel de aislamiento para operaciones criticas de StaffTrainingPlatform.

## Criterio junior

Un junior debe conocer los niveles de aislamiento de SQL Server y entender que Read Committed es el default.

## Criterio mid

Un mid debe reconocer cuando el locking pesimista por defecto de SQL Server causa contencion que Postgres no tendria, y evaluar Snapshot Isolation como alternativa.

## Criterio senior

Un senior debe decidir la estrategia de aislamiento de un sistema completo, considerando activar Snapshot Isolation para reducir contencion en escenarios de alta concurrencia.

## Errores comunes

- Asumir que SQL Server se comporta identico a Postgres por defecto, sorprendiendose de bloqueos entre lectores y escritores que no ocurririan en Postgres con MVCC.
- Usar Read Uncommitted por default para "mejorar performance", aceptando dirty reads sin entender el riesgo real de datos incorrectos.
- No considerar Snapshot Isolation cuando la contencion de locking tradicional es un problema real medido.
- No documentar la eleccion de nivel de aislamiento para operaciones criticas.

## Practica

- [ ] Configura distintos niveles de aislamiento y observa el comportamiento de bloqueo entre transacciones concurrentes.
- [ ] Activa Snapshot Isolation en una base de datos de practica y compara el comportamiento con Read Committed default.
- [ ] Simula el problema de contencion de locking tradicional que Snapshot Isolation resolveria.
- [ ] Compara explicitamente el comportamiento por defecto de SQL Server con el de Postgres ya estudiado.
- [ ] Documenta cuando activarias Snapshot Isolation para un sistema real.

## Preguntas de repaso

- Que diferencia fundamental de implementacion hay entre el aislamiento por defecto de SQL Server y de Postgres?
- Que hace Read Uncommitted y por que generalmente se desaconseja?
- Como se activa Snapshot Isolation y que comportamiento acerca al de Postgres?
- Cuando evaluarias activar Snapshot Isolation en un sistema de produccion?
