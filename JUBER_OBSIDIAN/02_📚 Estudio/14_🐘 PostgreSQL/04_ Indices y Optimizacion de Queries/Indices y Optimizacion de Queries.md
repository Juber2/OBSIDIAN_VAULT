---
tags:
  - PostgreSQL
  - Performance
  - Postgres-Indices
Creado: 2026-07-29
Relacionado:
  - POSTGRESQL
---

# Indices y Optimizacion de Queries

## Proposito del capitulo

Este capitulo cubre optimizacion de queries en Postgres de forma completa: tipos de indices (B-tree, Hash, GIN, GiST), lectura profunda de EXPLAIN ANALYZE, indices compuestos y parciales, funcionamiento del query planner, y el problema de N+1 desde la perspectiva SQL pura.

**Nivel esperado:** Mid a Senior.

## Temas

| Orden | Tema | Estado | Nota |
|---|---|---|---|
| 1 | Tipos de Indices B-tree Hash GIN GiST | Pendiente | [[01 - Tipos de Indices B-tree Hash GIN GiST]] |
| 2 | EXPLAIN ANALYZE | Pendiente | [[02 - EXPLAIN ANALYZE]] |
| 3 | Indices Compuestos y Parciales | Pendiente | [[03 - Indices Compuestos y Parciales]] |
| 4 | Query Planner | Pendiente | [[04 - Query Planner]] |
| 5 | N Mas 1 y Optimizacion de Joins | Pendiente | [[05 - N Mas 1 y Optimizacion de Joins]] |

## Como cerrar este capitulo

- [ ] Lei cada tema como capitulo, no solo como checklist.
- [ ] Optimice una query lenta usando `EXPLAIN ANALYZE` como evidencia.
- [ ] Diseñe un indice compuesto con el orden de columnas correcto.
- [ ] Detecte y corregi un caso de N+1 real contando queries SQL.
- [ ] Tengo notas propias de dudas, decisiones o patrones que quiero repetir.
