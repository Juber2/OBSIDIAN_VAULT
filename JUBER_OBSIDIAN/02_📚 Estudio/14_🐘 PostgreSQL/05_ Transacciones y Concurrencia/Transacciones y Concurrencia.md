---
tags:
  - PostgreSQL
  - Transacciones
  - Postgres-Transacciones
Creado: 2026-07-29
Relacionado:
  - POSTGRESQL
---

# Transacciones y Concurrencia

## Proposito del capitulo

Este capitulo cubre el corazon de la concurrencia en Postgres: ACID aplicado al motor, MVCC en profundidad, niveles de aislamiento, locking explicito a nivel de fila, y estrategias de prevencion de deadlocks.

**Nivel esperado:** Mid a Senior.

## Temas

| Orden | Tema | Estado | Nota |
|---|---|---|---|
| 1 | ACID en Postgres | Pendiente | [[01 - ACID en Postgres]] |
| 2 | MVCC en Profundidad | Pendiente | [[02 - MVCC en Profundidad]] |
| 3 | Niveles de Aislamiento | Pendiente | [[03 - Niveles de Aislamiento]] |
| 4 | Locks Row-Level y Table-Level | Pendiente | [[04 - Locks Row-Level y Table-Level]] |
| 5 | Deadlocks y su Prevencion | Pendiente | [[05 - Deadlocks y su Prevencion]] |

## Como cerrar este capitulo

- [ ] Lei cada tema como capitulo, no solo como checklist.
- [ ] Puedo explicar MVCC y por que genera la necesidad de VACUUM.
- [ ] Se elegir el nivel de aislamiento apropiado para una operacion critica.
- [ ] Implemente `FOR UPDATE` y logica de reintento ante deadlocks.
- [ ] Tengo notas propias de dudas, decisiones o patrones que quiero repetir.
