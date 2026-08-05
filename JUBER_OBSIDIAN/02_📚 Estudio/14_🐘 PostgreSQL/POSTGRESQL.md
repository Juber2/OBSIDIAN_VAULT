---
tags:
  - PostgreSQL
  - Bases-de-Datos
  - Backend
  - Libro
Creado: 2026-07-29
Relacionado:
  - ESTUDIO
---

#Principal

# Libro PostgreSQL Junior a Senior

Esta carpeta no es solo una ruta. Esta pensada como un **libro de estudio practico** para avanzar desde fundamentos de SQL hasta criterio senior de administracion y arquitectura de PostgreSQL.

La estructura esta dividida en capitulos. Cada capitulo contiene explicacion, modelo mental, ejemplos, errores comunes, practica, preguntas de repaso y criterios junior/mid/senior.

## Capitulos

| Orden | Capitulo | Estado | Nota |
|---|---|---|---|
| 1 | Fundamentos de PostgreSQL | Pendiente | [[Fundamentos de PostgreSQL]] |
| 2 | SQL Fundamentals en Postgres | Pendiente | [[SQL Fundamentals en Postgres]] |
| 3 | Modelado de Datos y Diseño de Esquemas | Pendiente | [[Modelado de Datos y Diseño de Esquemas]] |
| 4 | Indices y Optimizacion de Queries | Pendiente | [[Indices y Optimizacion de Queries]] |
| 5 | Transacciones y Concurrencia | Pendiente | [[Transacciones y Concurrencia]] |
| 6 | Funciones Vistas y Procedimientos | Pendiente | [[Funciones Vistas y Procedimientos]] |
| 7 | Tipos de Datos Avanzados | Pendiente | [[Tipos de Datos Avanzados]] |
| 8 | Administracion y Mantenimiento | Pendiente | [[Administracion y Mantenimiento]] |
| 9 | Replicacion y Alta Disponibilidad | Pendiente | [[Replicacion y Alta Disponibilidad]] |
| 10 | Postgres en la Nube y Contenedores | Pendiente | [[Postgres en la Nube y Contenedores]] |
| 11 | Postgres con NestJS y TypeORM | Pendiente | [[Postgres con NestJS y TypeORM]] |
| 12 | Performance Avanzada | Pendiente | [[Performance Avanzada]] |
| 13 | Senior PostgreSQL | Pendiente | [[Senior PostgreSQL]] |
| 14 | Proyectos PostgreSQL | Pendiente | [[Proyectos]] |

## Como usar este libro

1. Lee el capitulo en orden.
2. Ejecuta cada ejemplo en un Postgres real (local, Docker, o servicio cloud gratuito).
3. Conecta cada tema con [[NESTJS|el backend NestJS]] cuando corresponda, especialmente en el capitulo 11.
4. Convierte cada tema importante en un cambio real sobre un esquema de practica.
5. Vuelve a cada nota y marca el checklist solo cuando puedas explicarlo sin mirar tutorial.

## Resultado esperado

Al terminar, deberias poder diseñar, optimizar, asegurar y operar bases de datos PostgreSQL reales: SQL solido, modelado relacional correcto, indices basados en evidencia, transacciones y concurrencia con MVCC, programabilidad con funciones/triggers, tipos avanzados (JSONB, arrays, Full Text Search), administracion y backup confiables, alta disponibilidad con replicacion, integracion profunda con NestJS/TypeORM, y performance de nivel de produccion.

## Checkpoint senior

- [ ] Puedo modelar esquemas normalizados con constraints que replican reglas de negocio como defensa en profundidad.
- [ ] Domino EXPLAIN ANALYZE y diseño indices basados en evidencia, no intuicion.
- [ ] Puedo elegir el nivel de aislamiento apropiado y prevenir deadlocks por diseño.
- [ ] Puedo aprovechar JSONB, arrays y Full Text Search sin abusar de ellos como escape hatch de mal modelado.
- [ ] Puedo diseñar backups probados, PITR, y una topologia de replicacion con criterio real.
- [ ] Puedo integrar Postgres con NestJS/TypeORM aprovechando features especificas del motor de forma segura.
- [ ] Puedo decidir con criterio arquitectonico particionamiento, RLS, y migraciones zero-downtime en produccion.

## Referencias base

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL Wiki - Performance](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [PGTune](https://pgtune.leopard.in.ua/)
- [Use The Index, Luke](https://use-the-index-luke.com/)
