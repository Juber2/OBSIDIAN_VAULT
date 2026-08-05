---
tags:
  - PostgreSQL
  - Proyectos
  - Postgres-Proyectos
Creado: 2026-07-29
Relacionado:
  - Proyectos
  - POSTGRESQL
---

# 01 - Proyecto Esquema Completo Normalizado

## Que vas a construir

Un esquema relacional completo para un dominio real (por ejemplo, un sistema de reservas o un e-commerce simple), aplicando de punta a punta los capitulos de SQL Fundamentals y Modelado de Datos.

## Alcance funcional

- Al menos cinco tablas relacionadas, normalizadas a 3FN.
- Relaciones de los tres tipos (1a1, 1aN, NaM) correctamente modeladas con sus constraints apropiados.
- Columnas de auditoria estandar (`creado_en`, `actualizado_en`) en cada tabla.
- Constraints de integridad (`CHECK`, `UNIQUE`, `NOT NULL`) replicando al menos tres reglas de negocio reales.
- Datos de ejemplo insertados y consultados con JOINs y agregaciones.

## Checklist tecnica

- [ ] El esquema esta normalizado a 3FN, sin redundancia no justificada.
- [ ] Cada clave foranea tiene una politica `ON DELETE` explicita y justificada.
- [ ] Al menos una relacion muchos a muchos esta modelada con tabla intermedia correcta.
- [ ] Se ejecutaron consultas con `JOIN`, `GROUP BY`/`HAVING` sobre datos de ejemplo reales.
- [ ] El esquema completo esta documentado (diagrama entidad-relacion o descripcion escrita).

## Criterio de exito

El proyecto se considera completo cuando el esquema resiste preguntas de negocio reales (reportes agregados, listados filtrados) resueltas con SQL puro, sin necesitar reestructurar tablas para responderlas, y cuando otra persona puede entender el modelo completo leyendo la documentacion.

## Errores comunes a evitar

- Modelar una relacion muchos a muchos con un array o columna JSON en vez de una tabla intermedia real.
- No definir politicas `ON DELETE` explicitas, dejando el comportamiento por defecto sin decision consciente.
- Saltar directamente a escribir SQL sin pasar por el proceso conceptual de identificar entidades y relaciones primero.

## Extension opcional

Conectar este esquema con una aplicacion NestJS real, mapeando cada tabla a una entidad TypeORM, como puente hacia el proyecto de integracion mas adelante en este capitulo.
