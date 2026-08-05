---
tags:
  - PostgreSQL
  - Modelado
  - Postgres-Modelado
Creado: 2026-07-29
Relacionado:
  - Modelado de Datos y Diseño de Esquemas
  - POSTGRESQL
---

# 04 - Relaciones 1a1 1aN NaM

## Que vas a aprender

En este capitulo vas a dominar el modelado de los tres tipos de relaciones entre tablas: **uno a uno**, **uno a muchos**, y **muchos a muchos** (via tabla intermedia), la base tecnica exacta de las relaciones de TypeORM ya usadas en el libro de NestJS.

## Explicacion

Una relacion **uno a muchos** (la mas comun) se modela con una clave foranea en la tabla del lado "muchos" apuntando a la tabla del lado "uno" (un cliente tiene muchos pedidos: la clave foranea `cliente_id` vive en `pedidos`). Una relacion **uno a uno** se modela igual, pero con un constraint `UNIQUE` adicional en la columna de clave foranea, garantizando que cada fila del lado "uno" tenga a lo sumo una fila relacionada (un usuario tiene un unico perfil extendido).

Una relacion **muchos a muchos** (un producto puede estar en muchos pedidos, y un pedido puede tener muchos productos) no puede modelarse con una unica clave foranea; requiere una **tabla intermedia** (tabla de union o "junction table") con dos claves foraneas, una hacia cada tabla relacionada, y tipicamente su propia clave primaria compuesta o generada.

## Modelo mental

Pensa una relacion uno a muchos como cada empleado (muchos) teniendo escrito en su ficha a que unico departamento pertenece (uno). Una relacion uno a uno es como cada persona teniendo exactamente un unico numero de documento de identidad, nunca compartido con otra. Una relacion muchos a muchos es como una lista de inscripciones de estudiantes a cursos: ni el estudiante "pertenece" a un unico curso, ni el curso "pertenece" a un unico estudiante, asi que se necesita una tabla separada que registre cada combinacion especifica de "este estudiante esta en este curso".

## Ejemplo base

```sql
-- Uno a muchos: un cliente tiene muchos pedidos
CREATE TABLE pedidos (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    cliente_id bigint NOT NULL REFERENCES clientes(id)
);

-- Uno a uno: un usuario tiene un unico perfil extendido
CREATE TABLE perfiles (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    usuario_id bigint NOT NULL UNIQUE REFERENCES usuarios(id),
    biografia text
);

-- Muchos a muchos: productos y pedidos via tabla intermedia
CREATE TABLE pedido_productos (
    pedido_id bigint NOT NULL REFERENCES pedidos(id),
    producto_id bigint NOT NULL REFERENCES productos(id),
    cantidad integer NOT NULL DEFAULT 1,
    PRIMARY KEY (pedido_id, producto_id)   -- clave primaria compuesta
);
```

## Como se usa en un proyecto real

1. Se modela cada relacion uno a muchos poniendo la clave foranea en el lado correcto (siempre en el lado "muchos"), un error de diseño comun al invertirlo.
2. Se agrega `UNIQUE` a la clave foranea cuando la relacion realmente es uno a uno, evitando modelarla accidentalmente como uno a muchos sin esa restriccion.
3. Se crea una tabla intermedia con clave primaria compuesta para toda relacion muchos a muchos, agregando columnas adicionales (como `cantidad`) cuando la relacion misma tiene datos propios.
4. Se conecta esto directamente con los decoradores `@OneToMany`, `@OneToOne`, `@ManyToMany` de TypeORM ya vistos en el libro de NestJS, entendiendo el SQL exacto que genera cada uno.
5. Se documenta el diagrama de relaciones de un esquema complejo (entidad-relacion) como parte de la documentacion tecnica del proyecto.

## Criterio junior

Un junior debe poder modelar una relacion uno a muchos basica, y reconocer cuando una relacion muchos a muchos necesita una tabla intermedia.

## Criterio mid

Un mid debe modelar correctamente relaciones uno a uno con `UNIQUE`, y diseñar tablas intermedias con columnas adicionales cuando la relacion tiene datos propios.

## Criterio senior

Un senior debe diseñar el modelo relacional completo de un sistema complejo, revisando en code review que cada tipo de relacion este modelado correctamente y conectandolo con las entidades TypeORM equivalentes.

## Errores comunes

- Poner la clave foranea en el lado equivocado de una relacion uno a muchos, invirtiendo accidentalmente la relacion.
- Modelar una relacion uno a uno sin el constraint `UNIQUE`, permitiendo sin querer que se convierta en uno a muchos.
- Intentar modelar una relacion muchos a muchos con una unica clave foranea, sin usar una tabla intermedia.
- No agregar una clave primaria (compuesta o propia) a la tabla intermedia, permitiendo duplicados de la misma combinacion de relacion.

## Practica

- [ ] Modela una relacion uno a muchos entre dos tablas de tu eleccion.
- [ ] Modela una relacion uno a uno con el constraint `UNIQUE` correspondiente.
- [ ] Modela una relacion muchos a muchos con una tabla intermedia, incluyendo una columna adicional propia de la relacion.
- [ ] Conecta cada relacion modelada con el decorador TypeORM equivalente (`@OneToMany`, `@OneToOne`, `@ManyToMany`).
- [ ] Dibuja (o describe) el diagrama entidad-relacion de un esquema real de tu proyecto.

## Preguntas de repaso

- En que lado de una relacion uno a muchos deberia vivir la clave foranea?
- Que constraint adicional convierte una relacion uno a muchos en uno a uno?
- Por que una relacion muchos a muchos necesita una tabla intermedia?
- Como se relaciona cada tipo de relacion SQL con los decoradores de TypeORM vistos en el libro de NestJS?
