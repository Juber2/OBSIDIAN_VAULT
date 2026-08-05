---
tags:
  - PostgreSQL
  - Bases-de-Datos
  - Postgres-Fundamentos
Creado: 2026-07-29
Relacionado:
  - Fundamentos de PostgreSQL
  - POSTGRESQL
---

# 01 - Que es PostgreSQL y por que Elegirlo

## Que vas a aprender

En este capitulo vas a entender **que es PostgreSQL**, su filosofia como base de datos relacional **objeto-relacional** de codigo abierto, y por que se volvio la eleccion por defecto para proyectos nuevos en el ecosistema Node/NestJS que ya conoces.

## Explicacion

PostgreSQL (o simplemente "Postgres") es un sistema de gestion de bases de datos relacional, de codigo abierto, con mas de 25 años de desarrollo continuo. A diferencia de motores mas simples, Postgres es **objeto-relacional**: ademas del modelo tabular clasico, soporta tipos de datos complejos (arrays, JSON/JSONB, tipos geometricos), herencia de tablas, y un sistema de extensiones que le permite agregar capacidades completas (como busqueda de texto o datos geoespaciales con PostGIS) sin salir del motor.

Su reputacion de **estricta adherencia al estandar SQL** y su enfoque en correccion de datos (constraints reales, integridad referencial estricta) lo hicieron la eleccion favorita en el ecosistema de TypeORM/Prisma (ya visto en el libro de NestJS), donde la mayoria de tutoriales, ORMs y plataformas cloud (Supabase, Neon, RDS) lo tratan como el motor relacional por defecto.

## Modelo mental

Pensa Postgres como una biblioteca extremadamente rigurosa con su bibliotecario: no te deja archivar un libro fuera de lugar (constraints estrictos), pero ademas de libros tradicionales (tablas relacionales), puede archivar mapas (datos geoespaciales), documentos con estructura variable (JSONB), y crear indices de busqueda por contenido (full text search), todo dentro del mismo sistema.

## Ejemplo base

```sql
SELECT version();
-- PostgreSQL 16.x on x86_64-pc-linux-gnu...

\l          -- lista bases de datos (en psql)
\dt          -- lista tablas de la base actual
```

## Como se usa en un proyecto real

1. Se elige Postgres por defecto para proyectos nuevos con NestJS/TypeORM o Prisma, salvo una razon especifica que justifique otro motor.
2. Se aprovechan sus tipos avanzados (JSONB, arrays) cuando el modelo de datos lo justifica, sin abusar de ellos para evitar diseño relacional apropiado.
3. Se evalua la disponibilidad de extensiones necesarias (PostGIS, pg_trgm, uuid-ossp) antes de elegir un proveedor cloud especifico, dado que no todos las soportan igual.
4. Se documenta la version de Postgres usada en cada ambiente, dado que features nuevas (como mejoras de particionamiento) varian entre versiones mayores.
5. Se conecta esta eleccion con el capitulo de Base de Datos del libro de NestJS, donde TypeORM y Prisma ya se usaron especificamente sobre Postgres.

## Criterio junior

Un junior debe poder explicar que es Postgres y ejecutar comandos basicos de exploracion en `psql`.

## Criterio mid

Un mid debe justificar cuando Postgres es la eleccion apropiada frente a otros motores relacionales, considerando el ecosistema de herramientas disponible.

## Criterio senior

Un senior debe decidir la estrategia de base de datos de un sistema completo, evaluando Postgres frente a alternativas segun requisitos reales (tipos de datos, extensiones, ecosistema de la organizacion).

## Errores comunes

- Elegir Postgres "porque es lo que se usa siempre" sin evaluar si el proyecto tiene requisitos que otro motor resolveria mejor.
- No verificar que extensiones necesarias estan disponibles en el proveedor cloud elegido antes de comprometerse con esa plataforma.
- Subestimar el rigor de Postgres con constraints, generando errores de insercion que en un motor mas laxo pasarian desapercibidos.
- No documentar la version de Postgres usada, generando sorpresas al migrar entre ambientes con versiones distintas.

## Practica

- [ ] Instala Postgres (local, Docker, o servicio cloud gratuito) y conectate con `psql`.
- [ ] Explora las bases de datos y tablas existentes con `\l` y `\dt`.
- [ ] Verifica la version instalada con `SELECT version();`.
- [ ] Investiga que extensiones utiles ofrece tu instalacion con `\dx`.
- [ ] Anota por que elegirias (o no) Postgres para un proyecto nuevo tuyo.

## Preguntas de repaso

- Que significa que Postgres sea "objeto-relacional" en vez de puramente relacional?
- Que ventaja da el sistema de extensiones de Postgres frente a un motor sin esa capacidad?
- Por que Postgres se volvio el motor por defecto en el ecosistema de TypeORM/Prisma?
- Que verificarias antes de elegir un proveedor cloud especifico para alojar Postgres?
