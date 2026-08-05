---
tags:
  - PostgreSQL
  - SQL
  - Postgres-SQL
Creado: 2026-07-29
Relacionado:
  - SQL Fundamentals en Postgres
  - POSTGRESQL
---

# 01 - CREATE TABLE y Tipos de Datos

## Que vas a aprender

En este capitulo vas a aprender a crear tablas con `CREATE TABLE`, y los tipos de datos fundamentales de Postgres: numericos, texto, fecha/hora, booleanos, y el tipo `serial`/`identity` para autoincrementales.

## Explicacion

`CREATE TABLE` define el nombre de la tabla y sus columnas, cada una con un tipo de dato y opcionalmente restricciones (`NOT NULL`, `DEFAULT`, ver capitulo de Modelado para constraints completos). Postgres tiene tipos numericos precisos: `integer` (32 bits), `bigint` (64 bits, recomendado para IDs en tablas que creceran mucho), `numeric(precision, escala)` para valores monetarios exactos (nunca `float` para dinero, dado que los tipos de punto flotante tienen imprecision inherente).

Para texto, `text` (longitud ilimitada) es preferido sobre `varchar(n)` en Postgres a menos que se necesite un limite estricto real, dado que a diferencia de otros motores, no hay penalidad de performance por usar `text`. Para identificadores autoincrementales, la sintaxis moderna recomendada es `GENERATED ALWAYS AS IDENTITY` (estandar SQL) en vez del `serial` legacy, aunque ambos siguen siendo comunes en codigo existente.

## Modelo mental

Pensa `CREATE TABLE` como diseñar el formulario oficial de un tipo de registro: cada campo (columna) tiene un tipo de dato que define que forma de informacion acepta, similar a como un formulario en papel tiene casillas especificas para numeros, texto, o fechas, rechazando lo que no encaja en esa casilla.

## Ejemplo base

```sql
CREATE TABLE usuarios (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email text NOT NULL,
    nombre text NOT NULL,
    edad integer,
    saldo numeric(10, 2) DEFAULT 0,
    activo boolean DEFAULT true,
    creado_en timestamptz DEFAULT now()
);

-- Legacy pero aun comun
CREATE TABLE productos (
    id serial PRIMARY KEY,
    nombre varchar(255) NOT NULL,
    precio numeric(10, 2) NOT NULL
);
```

## Como se usa en un proyecto real

1. Se usa `bigint GENERATED ALWAYS AS IDENTITY` para IDs autoincrementales en tablas nuevas, evitando el riesgo de agotar el rango de `integer` en tablas de alto volumen a largo plazo.
2. Se usa `numeric` (nunca `float`/`double precision`) para cualquier valor monetario, evitando errores de redondeo acumulados con el tiempo.
3. Se usa `text` en vez de `varchar(n)` salvo que exista un limite de negocio real que valga la pena forzar a nivel de base de datos.
4. Se usa `timestamptz` (con zona horaria) en vez de `timestamp` sin zona, evitando ambiguedad al trabajar con usuarios en distintas zonas horarias.
5. Se conecta esto con las entidades TypeORM ya vistas en el libro de NestJS, donde estos mismos tipos se mapean a decoradores de columna especificos.

## Criterio junior

Un junior debe poder crear una tabla simple con columnas de tipos basicos (texto, numero, fecha, booleano).

## Criterio mid

Un mid debe elegir el tipo numerico correcto segun el caso (integer vs bigint vs numeric) y usar `timestamptz` correctamente para datos de fecha/hora.

## Criterio senior

Un senior debe definir convenciones de tipos de datos para todo el equipo (cuando usar `text` vs `varchar`, `serial` vs `identity`), documentando la justificacion.

## Errores comunes

- Usar `float`/`double precision` para valores monetarios, arriesgando errores de redondeo acumulados con el tiempo.
- Usar `timestamp` sin zona horaria, generando ambiguedad y bugs sutiles al trabajar con usuarios en distintas zonas horarias.
- Usar `integer` para IDs de tablas que crecen indefinidamente, arriesgando agotar el rango a largo plazo.
- Restringir innecesariamente con `varchar(n)` sin un limite de negocio real que lo justifique.

## Practica

- [ ] Crea una tabla con columnas de tipos numericos, texto, fecha y booleano.
- [ ] Compara `serial` con `GENERATED ALWAYS AS IDENTITY` para una columna de ID.
- [ ] Crea una columna monetaria con `numeric` y verifica que no pierde precision en operaciones.
- [ ] Usa `timestamptz` y verifica como se comporta al insertar desde distintas zonas horarias simuladas.
- [ ] Conecta el tipo de cada columna con el decorador TypeORM equivalente que usarias en NestJS.

## Preguntas de repaso

- Por que `numeric` es el tipo correcto para valores monetarios y no `float`?
- Que ventaja tiene `bigint` sobre `integer` para IDs de tablas de alto volumen?
- Por que `text` es preferido sobre `varchar(n)` en Postgres cuando no hay un limite real de negocio?
- Que diferencia hay entre `timestamp` y `timestamptz`?
