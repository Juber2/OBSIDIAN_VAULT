---
tags:
  - PostgreSQL
  - Performance
  - Postgres-Performance-Avanzada
Creado: 2026-07-29
Relacionado:
  - Performance Avanzada
  - POSTGRESQL
---

# 01 - Particionamiento de Tablas

## Que vas a aprender

En este capitulo vas a aprender **particionamiento de tablas** (`PARTITION BY`), la tecnica para dividir una tabla logicamente unica en multiples tablas fisicas mas chicas, mejorando drasticamente performance en tablas con volumenes muy grandes de datos.

## Explicacion

Cuando una tabla crece a decenas o cientos de millones de filas, incluso con buenos indices (ver capitulo de Indices), operaciones de mantenimiento (VACUUM, ver capitulo de Administracion) y ciertas consultas pueden volverse costosas simplemente por el tamaño fisico de la tabla. El **particionamiento** divide la tabla en particiones fisicas mas pequeñas segun una estrategia: **RANGE** (por rango de valores, tipico para datos temporales: una particion por mes), **LIST** (por valores discretos especificos), o **HASH** (distribucion uniforme sin criterio de negocio particular).

Postgres presenta las particiones como si fueran una unica tabla logica (**partition pruning**: el planner automaticamente descarta particiones irrelevantes segun el filtro de la consulta, consultando solo las particiones necesarias), pero internamente cada particion es una tabla fisica independiente que puede tener sus propios indices, y operaciones de mantenimiento como eliminar datos viejos se vuelven instantaneas (`DROP TABLE` de una particion completa, en vez de un `DELETE` masivo lento con generacion de tuplas muertas, ver capitulo de MVCC).

## Modelo mental

Pensa una tabla particionada como un archivo de documentos organizado en cajones separados por año, en vez de un unico cajon gigante con todos los documentos de todos los años mezclados. Buscar un documento de un año especifico solo requiere abrir ese cajon (partition pruning), no revisar el archivo completo. Y cuando un año completo de documentos ya no se necesita, simplemente se tira el cajon entero (drop de la particion), en vez de sacar cada documento uno por uno.

## Ejemplo base

```sql
CREATE TABLE eventos (
    id bigint GENERATED ALWAYS AS IDENTITY,
    tipo text,
    creado_en timestamptz NOT NULL,
    PRIMARY KEY (id, creado_en)
) PARTITION BY RANGE (creado_en);

CREATE TABLE eventos_2026_07 PARTITION OF eventos
    FOR VALUES FROM ('2026-07-01') TO ('2026-08-01');

CREATE TABLE eventos_2026_08 PARTITION OF eventos
    FOR VALUES FROM ('2026-08-01') TO ('2026-09-01');

-- El planner automaticamente solo revisa la particion relevante (partition pruning)
EXPLAIN ANALYZE SELECT * FROM eventos WHERE creado_en >= '2026-07-15' AND creado_en < '2026-07-20';

-- Eliminar datos viejos: instantaneo, sin generar tuplas muertas
DROP TABLE eventos_2026_07;
```

## Como se usa en un proyecto real

1. Se evalua particionamiento cuando una tabla realmente alcanza un volumen que justifica la complejidad adicional (generalmente decenas de millones de filas o mas), no como optimizacion prematura.
2. Se usa particionamiento RANGE por fecha para tablas de eventos, logs, o auditoria, donde datos viejos se archivan o eliminan periodicamente completos.
3. Se aprovecha `DROP TABLE` de una particion completa para eliminar datos viejos instantaneamente, en vez de un `DELETE` masivo lento que genera tuplas muertas (ver capitulo de MVCC).
4. Se automatiza la creacion de nuevas particiones (mensual, por ejemplo) con un job programado (ver cron en el libro de Linux), evitando que se olvide crear la particion del proximo periodo.
5. Se verifica con `EXPLAIN ANALYZE` que el partition pruning realmente esta funcionando, consultando solo las particiones relevantes segun el filtro usado.

## Criterio junior

Un junior debe entender el concepto de particionamiento y poder crear una tabla particionada por RANGE basica.

## Criterio mid

Un mid debe automatizar la creacion de nuevas particiones periodicamente, y verificar partition pruning con `EXPLAIN ANALYZE`.

## Criterio senior

Un senior debe decidir cuando el particionamiento realmente se justifica para una tabla especifica, diseñar la estrategia de particion apropiada, y automatizar completamente el ciclo de vida de particiones (creacion y archivado/eliminacion).

## Errores comunes

- Particionar una tabla que no tiene volumen suficiente para justificar la complejidad adicional, agregando overhead sin beneficio real.
- No automatizar la creacion de nuevas particiones, descubriendo el problema cuando datos nuevos no tienen particion donde caer.
- Usar `DELETE` masivo en vez de `DROP TABLE` de particiones completas para eliminar datos viejos, perdiendo la principal ventaja de mantenimiento del particionamiento.
- No verificar que el partition pruning realmente funciona para los patrones de consulta reales de la aplicacion.

## Practica

- [ ] Crea una tabla particionada por RANGE de fecha con al menos dos particiones.
- [ ] Verifica con `EXPLAIN ANALYZE` que una consulta con filtro de fecha solo revisa la particion relevante.
- [ ] Simula la eliminacion de datos viejos con `DROP TABLE` de una particion completa.
- [ ] Diseña como automatizarias la creacion periodica de nuevas particiones.
- [ ] Evalua si una tabla real de tu proyecto se beneficiaria de particionamiento.

## Preguntas de repaso

- Que problema de performance resuelve el particionamiento en tablas muy grandes?
- Que es partition pruning y como lo aprovecha el query planner automaticamente?
- Por que `DROP TABLE` de una particion es mucho mas eficiente que un `DELETE` masivo equivalente?
- Cuando NO se justificaria particionar una tabla?
