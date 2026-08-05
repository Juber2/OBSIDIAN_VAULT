---
tags:
  - SQLServer
  - Transacciones
  - SQLServer-Transacciones
Creado: 2026-07-29
Relacionado:
  - SQL SERVER
---

# Transacciones y Concurrencia

## Proposito del capitulo

Este capitulo cubre concurrencia en SQL Server: ACID aplicado a T-SQL, niveles de aislamiento y su diferencia critica con Postgres (locking vs MVCC por defecto), lock escalation especifico de SQL Server, deadlocks, y Snapshot Isolation/RCSI como forma de acercarse al modelo ya conocido de Postgres.

**Nivel esperado:** Mid a Senior.

## Temas

| Orden | Tema | Estado | Nota |
|---|---|---|---|
| 1 | ACID en SQL Server | Pendiente | [[01 - ACID en SQL Server]] |
| 2 | Niveles de Aislamiento | Pendiente | [[02 - Niveles de Aislamiento]] |
| 3 | Locks y Lock Escalation | Pendiente | [[03 - Locks y Lock Escalation]] |
| 4 | Deadlocks | Pendiente | [[04 - Deadlocks]] |
| 5 | Snapshot Isolation | Pendiente | [[05 - Snapshot Isolation]] |

## Como cerrar este capitulo

- [ ] Lei cada tema como capitulo, no solo como checklist.
- [ ] Puedo explicar por que SQL Server bloquea lectores contra escritores por defecto, a diferencia de Postgres.
- [ ] Provoque lock escalation y lo evite rediseñando en lotes.
- [ ] Active RCSI y comprendi su impacto en tempdb.
- [ ] Tengo notas propias de dudas, decisiones o patrones que quiero repetir.
