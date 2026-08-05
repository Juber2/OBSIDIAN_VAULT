---
tags:
  - PostgreSQL
  - Cloud
  - Postgres-Cloud
Creado: 2026-07-29
Relacionado:
  - Postgres en la Nube y Contenedores
  - POSTGRESQL
---

# 04 - Migraciones con Herramientas

## Que vas a aprender

En este capitulo vas a consolidar el manejo de **migraciones de esquema** con herramientas dedicadas, conectando directamente con lo ya visto en el capitulo de Migrations del libro de NestJS (TypeORM), y comparando con alternativas independientes del ecosistema como Flyway.

## Explicacion

Ya se vio en profundidad en el libro de NestJS por que las migraciones versionadas son el mecanismo seguro de evolucionar un esquema (nunca `synchronize: true` en produccion). Aca se profundiza en la mecanica desde el lado de Postgres: cada migration es un cambio de esquema con capacidad de revertirse (`up`/`down`), aplicado en orden secuencial, con una tabla de control (`migrations` en TypeORM) que registra que migrations ya se ejecutaron.

**Flyway** y **Liquibase** son herramientas de migracion independientes del framework de aplicacion (a diferencia de las migrations de TypeORM, atadas al ecosistema Node), utiles cuando multiples aplicaciones en distintos lenguajes comparten la misma base de datos, o cuando el equipo de DBAs prefiere gestionar migraciones de forma centralizada sin depender del framework de cada aplicacion cliente.

## Modelo mental

Pensa las migraciones de TypeORM como el sistema de control de versiones integrado en el editor de texto que ya usas (comodo porque esta ahi mismo, pero atado a esa herramienta especifica). Flyway/Liquibase son como un sistema de control de versiones independiente y universal (como Git) que cualquier editor o herramienta puede usar, util cuando varias aplicaciones distintas necesitan coordinarse sobre el mismo repositorio de cambios.

## Ejemplo base

```bash
# TypeORM (ya visto en el libro de NestJS)
npx typeorm migration:generate src/migrations/AddPhoneToUsers -d src/data-source.ts
npx typeorm migration:run -d src/data-source.ts
```

```sql
-- Flyway: convencion de nombres de archivo versionados
-- V1__crear_tabla_usuarios.sql
CREATE TABLE usuarios (id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY, email text);

-- V2__agregar_telefono.sql
ALTER TABLE usuarios ADD COLUMN telefono text;
```

```bash
flyway -url=jdbc:postgresql://localhost/mi_app -user=postgres migrate
flyway info    -- muestra que migrations se aplicaron y cuales estan pendientes
```

## Como se usa en un proyecto real

1. Se usan las migrations de TypeORM (ya vistas en el libro de NestJS) como default para proyectos donde una unica aplicacion NestJS es dueña completa del esquema.
2. Se evalua Flyway/Liquibase cuando multiples servicios o lenguajes distintos comparten responsabilidad sobre el mismo esquema de base de datos.
3. Se aplica siempre el mismo principio (visto en NestJS y reforzado aca): cada migration debe tener un `down` funcional, probado, antes de mergear.
4. Se integra la ejecucion de migraciones en el pipeline de CI/CD como paso automatizado, aplicando el mismo patron ya visto en el libro de Docker (capitulo de CI/CD).
5. Se documenta claramente que herramienta de migracion usa cada proyecto y por que, especialmente en sistemas con multiples equipos o servicios.

## Criterio junior

Un junior debe poder generar y ejecutar una migration con TypeORM, y entender el concepto general que comparte con Flyway/Liquibase.

## Criterio mid

Un mid debe decidir cuando una herramienta independiente del framework (Flyway/Liquibase) se justifica sobre las migrations integradas de TypeORM.

## Criterio senior

Un senior debe diseñar la estrategia de gestion de esquema para sistemas con multiples servicios/lenguajes compartiendo una base de datos, coordinando responsabilidades claras sobre quien puede modificar que parte del esquema.

## Errores comunes

- Usar migrations de TypeORM cuando en realidad multiples aplicaciones en lenguajes distintos necesitan coordinarse sobre el mismo esquema, generando conflictos de responsabilidad.
- No mantener un `down` funcional en cada migration, perdiendo la capacidad de revertir un cambio problematico rapidamente.
- No integrar la ejecucion de migraciones en el pipeline de CI/CD, dependiendo de ejecucion manual propensa a olvidos.
- No documentar que herramienta de migracion gestiona cada esquema compartido, generando confusion sobre responsabilidades en sistemas complejos.

## Practica

- [ ] Genera y ejecuta una migration con TypeORM sobre un proyecto NestJS de practica, repasando lo ya visto en ese libro.
- [ ] Investiga la sintaxis basica de Flyway o Liquibase y compara con el enfoque de TypeORM.
- [ ] Evalua un caso hipotetico donde multiples servicios comparten una base de datos, y decide que herramienta de migracion usarias.
- [ ] Verifica que un `down` de una migration existente realmente revierte el cambio correctamente.
- [ ] Documenta la estrategia de migraciones de un proyecto real tuyo.

## Preguntas de repaso

- Que ventaja da una herramienta de migracion independiente del framework (Flyway/Liquibase) sobre las migrations integradas de TypeORM?
- Por que cada migration deberia tener un `down` funcional y probado?
- Cuando se justificaria migrar de TypeORM migrations hacia Flyway/Liquibase en un sistema real?
- Como integrarias la ejecucion de migraciones en un pipeline de CI/CD, aplicando lo ya visto en el libro de Docker?
