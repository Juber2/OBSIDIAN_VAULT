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

# 05 - Bases de Datos Esquemas y Catalogos

## Que vas a aprender

En este capitulo vas a entender la jerarquia de organizacion de Postgres: **cluster** (instancia del servidor), **bases de datos**, **esquemas** (schemas), y las **tablas del catalogo del sistema** donde Postgres guarda metadata sobre si mismo.

## Explicacion

Un unico **cluster** de Postgres (una instancia del servidor corriendo) puede contener multiples **bases de datos** completamente aisladas entre si (no se puede hacer un JOIN entre tablas de bases de datos distintas directamente). Dentro de cada base de datos, los **esquemas** (`schema`) son un nivel adicional de organizacion, como carpetas dentro de la base: por defecto todo vive en el esquema `public`, pero se pueden crear esquemas adicionales para separar logicamente modulos de una aplicacion, o para dar aislamiento entre distintos clientes en una arquitectura **multi-tenant** (ver capitulo relacionado en el libro de NestJS, donde se vio aislamiento multi-tenant por columna; esquemas separados son otra estrategia posible).

Postgres mantiene su propia metadata (que tablas existen, sus columnas, indices, constraints) en el **catalogo del sistema** (`information_schema`, el estandar SQL, y `pg_catalog`, especifico de Postgres con mas detalle interno), consultable con SQL normal, lo cual es la base tecnica de lo que meta-comandos como `\dt` hacen internamente.

## Modelo mental

Pensa el cluster como el edificio completo de Postgres, cada base de datos como un departamento independiente dentro de ese edificio (con paredes que no se pueden cruzar directamente), y los esquemas como habitaciones dentro de cada departamento, cada una pudiendo organizar sus propios muebles (tablas) sin mezclarse con las de otra habitacion, aunque esten en el mismo departamento.

## Ejemplo base

```sql
CREATE DATABASE mi_app;
\c mi_app

CREATE SCHEMA ventas;
CREATE SCHEMA inventario;

CREATE TABLE ventas.pedidos (id serial PRIMARY KEY, total numeric);
CREATE TABLE inventario.productos (id serial PRIMARY KEY, nombre text);

SET search_path TO ventas, public;   -- orden de busqueda de esquemas sin prefijo

-- Consultar el catalogo directamente
SELECT table_name FROM information_schema.tables WHERE table_schema = 'ventas';
```

## Como se usa en un proyecto real

1. Se usa el esquema `public` para la mayoria de aplicaciones simples, sin necesitar la complejidad adicional de multiples esquemas.
2. Se evaluan esquemas separados como estrategia de aislamiento multi-tenant cuando el numero de tenants es manejable (a diferencia de la estrategia por columna `tenantId`, vista en el libro de NestJS, mas apropiada para muchos tenants).
3. Se consulta `information_schema` (estandar, portable entre motores) cuando se necesita compatibilidad con otros motores SQL, y `pg_catalog` cuando se necesita detalle especifico de Postgres.
4. Se documenta claramente la estrategia de esquemas de un proyecto, dado que agrega una capa de organizacion que un nuevo integrante debe entender.
5. Se conecta esto con el capitulo de multi-tenancy del libro de NestJS, comparando la estrategia de columna `tenantId` con la de esquemas separados por tenant.

## Criterio junior

Un junior debe entender la jerarquia cluster -> base de datos -> esquema -> tabla, y poder crear una tabla dentro de un esquema especifico.

## Criterio mid

Un mid debe consultar el catalogo del sistema con SQL para obtener metadata de tablas, y configurar `search_path` apropiadamente.

## Criterio senior

Un senior debe decidir la estrategia de esquemas para aislamiento multi-tenant cuando corresponda, comparandola con alternativas ya vistas en el libro de NestJS, y documentar esa decision como un ADR.

## Errores comunes

- Intentar hacer un JOIN directo entre tablas de bases de datos distintas dentro del mismo cluster, sin entender que estan completamente aisladas.
- No configurar `search_path` correctamente, generando errores de "tabla no encontrada" al usar nombres sin prefijo de esquema.
- Usar multiples esquemas sin una razon organizativa real, agregando complejidad innecesaria a un proyecto simple.
- No documentar la estrategia de esquemas elegida, dificultando el entendimiento de nuevos integrantes del equipo.

## Practica

- [ ] Crea una base de datos nueva y al menos dos esquemas dentro de ella.
- [ ] Crea tablas en esquemas distintos y consultalas con el prefijo completo.
- [ ] Configura `search_path` para acceder a una tabla sin prefijo explicito.
- [ ] Consulta `information_schema.tables` para listar tablas de un esquema especifico.
- [ ] Compara la estrategia de esquemas separados con la de columna `tenantId` vista en el libro de NestJS para multi-tenancy.

## Preguntas de repaso

- Por que no se puede hacer un JOIN directo entre tablas de bases de datos distintas en el mismo cluster?
- Que ventaja organizativa dan los esquemas dentro de una misma base de datos?
- Que diferencia hay entre `information_schema` y `pg_catalog`?
- Cuando elegirias esquemas separados como estrategia de multi-tenancy en vez de una columna `tenantId`?
