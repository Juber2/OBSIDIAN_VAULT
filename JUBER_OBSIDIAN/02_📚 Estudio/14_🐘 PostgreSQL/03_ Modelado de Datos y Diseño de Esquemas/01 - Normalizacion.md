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

# 01 - Normalizacion

## Que vas a aprender

En este capitulo vas a aprender las **formas normales** (1FN, 2FN, 3FN) del diseño de bases de datos relacionales: que problema resuelve cada una, y cuando (y cuanto) normalizar en un proyecto real.

## Explicacion

La **normalizacion** es el proceso de organizar datos para minimizar redundancia y evitar anomalias de actualizacion. La **primera forma normal (1FN)** exige que cada columna contenga valores atomicos (no listas ni estructuras dentro de una celda) y que no haya grupos repetidos de columnas. La **segunda forma normal (2FN)** exige que cada columna no clave dependa de la clave primaria **completa** (relevante en claves compuestas). La **tercera forma normal (3FN)**, la mas comunmente aplicada en la practica, exige que no haya dependencias transitivas: una columna no clave no deberia depender de otra columna no clave.

En la practica, la mayoria de esquemas de aplicacion apuntan a 3FN como buen equilibrio, pero **desnormalizar deliberadamente** (duplicar datos a proposito) es una decision valida cuando la performance de lectura lo justifica, siempre como una decision consciente y documentada, no como resultado de no entender normalizacion en primer lugar.

## Modelo mental

Pensa la normalizacion como organizar un archivo de oficina para que cada dato importante viva en un unico lugar autorizado: si el telefono de un cliente esta escrito en veinte carpetas distintas (redundancia), actualizar ese telefono requiere encontrar y corregir las veinte copias, y es facil que alguna quede desactualizada. Normalizar es tener el telefono en un unico lugar, y que las otras carpetas simplemente lo referencien.

## Ejemplo base

```sql
-- Mal (no normalizado): datos del cliente repetidos en cada pedido
CREATE TABLE pedidos_mal (
    id serial PRIMARY KEY,
    cliente_nombre text,
    cliente_email text,
    producto text,
    total numeric
);

-- Bien (normalizado): datos del cliente en una unica tabla
CREATE TABLE clientes (
    id serial PRIMARY KEY,
    nombre text,
    email text
);

CREATE TABLE pedidos (
    id serial PRIMARY KEY,
    cliente_id integer REFERENCES clientes(id),
    total numeric
);
```

## Como se usa en un proyecto real

1. Se diseña el esquema inicial de cualquier proyecto apuntando a 3FN como default razonable, evitando redundancia sin justificacion.
2. Se identifican dependencias transitivas revisando si una columna no clave realmente depende de otra columna no clave en vez de la clave primaria.
3. Se documenta cualquier desnormalizacion deliberada (por ejemplo, guardar un total precalculado) explicando el tradeoff de performance aceptado.
4. Se conecta esto con el diseño de entidades TypeORM ya visto en el libro de NestJS, donde cada tabla normalizada corresponde a una entidad separada con sus relaciones.
5. Se revisa el diseño de esquema en code review buscando redundancia no intencional que indicaria un problema de normalizacion.

## Criterio junior

Un junior debe entender el problema que resuelve la normalizacion (evitar redundancia y anomalias de actualizacion) y reconocer un esquema no normalizado obvio.

## Criterio mid

Un mid debe diseñar esquemas normalizados a 3FN por defecto, identificando dependencias transitivas y separandolas en tablas apropiadas.

## Criterio senior

Un senior debe decidir con criterio cuando desnormalizar deliberadamente por razones de performance, documentando esa decision y sus tradeoffs para el equipo.

## Errores comunes

- Repetir datos que deberian vivir en una unica tabla (como datos de cliente en cada pedido), generando anomalias de actualizacion.
- Desnormalizar sin documentar la razon, dejando que futuros desarrolladores no entiendan por que existe esa redundancia deliberada.
- Sobre-normalizar hasta el punto de requerir demasiados joins para consultas comunes, sin considerar el impacto real de performance.
- No revisar el diseño de esquema en busca de dependencias transitivas antes de considerar el modelado terminado.

## Practica

- [ ] Identifica un esquema no normalizado (real o de ejemplo) con datos repetidos y redisenalo a 3FN.
- [ ] Explica con tus palabras que anomalia de actualizacion previene la normalizacion en ese caso.
- [ ] Diseña un caso donde desnormalizar deliberadamente seria una decision valida, documentando el tradeoff.
- [ ] Conecta un esquema normalizado con las entidades TypeORM correspondientes que usarias en NestJS.
- [ ] Revisa un esquema real de tu trabajo buscando redundancia no intencional.

## Preguntas de repaso

- Que problema concreto resuelve la normalizacion en terminos de mantenimiento de datos?
- Que exige la tercera forma normal (3FN) respecto a dependencias transitivas?
- Cuando seria una decision valida desnormalizar deliberadamente un esquema?
- Como conecta un esquema normalizado con el diseño de entidades en TypeORM?
