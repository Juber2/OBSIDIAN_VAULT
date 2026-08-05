---
tags:
  - PostgreSQL
  - Performance
  - Postgres-Indices
Creado: 2026-07-29
Relacionado:
  - Indices y Optimizacion de Queries
  - POSTGRESQL
---

# 01 - Tipos de Indices B-tree Hash GIN GiST

## Que vas a aprender

En este capitulo vas a aprender los distintos tipos de indices que ofrece Postgres: **B-tree** (el default, para la gran mayoria de casos), **Hash** (igualdad exacta), **GIN** (datos compuestos como JSONB y arrays), y **GiST** (busquedas geometricas y rangos).

## Explicacion

Un indice es una estructura de datos adicional que permite a Postgres encontrar filas sin recorrer la tabla completa (ya visto conceptualmente en el capitulo de Optimizacion del libro de NestJS). **B-tree** es el tipo por defecto (`CREATE INDEX` sin especificar tipo lo usa), eficiente para igualdad, rangos (`<`, `>`, `BETWEEN`), y ordenamiento, cubriendo la gran mayoria de casos de uso. **Hash** es mas eficiente solo para igualdad exacta pero no soporta rangos ni ordenamiento, siendo una opcion mas especializada y menos usada en la practica.

**GIN** (Generalized Inverted Index) es el indice correcto para columnas con datos compuestos: JSONB (buscar dentro de un documento), arrays (buscar si un elemento esta contenido), y full text search (ver capitulo de Tipos Avanzados). **GiST** (Generalized Search Tree) es la base de indices para datos geometricos y de rango (usado por PostGIS para datos geoespaciales, o para rangos de tiempo con exclusion de solapamiento).

## Modelo mental

Pensa un B-tree como el indice alfabetico tradicional de un libro: perfecto para buscar una palabra especifica o un rango ("todas las palabras entre M y P"). GIN es como un indice especial para libros que tienen contenido dentro de contenido (como una enciclopedia con articulos que mencionan otros articulos): te permite buscar "que articulos mencionan esta palabra en cualquier parte de su texto", algo que un indice alfabetico simple no puede resolver eficientemente.

## Ejemplo base

```sql
CREATE INDEX idx_usuarios_email ON usuarios (email);              -- B-tree, default
CREATE INDEX idx_pedidos_fecha ON pedidos USING hash (numero_orden); -- Hash, solo igualdad

CREATE INDEX idx_productos_metadata ON productos USING gin (metadata);  -- GIN para JSONB
CREATE INDEX idx_productos_tags ON productos USING gin (tags);           -- GIN para arrays

CREATE INDEX idx_eventos_rango ON eventos USING gist (periodo);          -- GiST para rangos
```

## Como se usa en un proyecto real

1. Se usa B-tree como default para la gran mayoria de columnas indexadas (claves foraneas, columnas de filtro y ordenamiento frecuente), sin necesitar especificar el tipo explicitamente.
2. Se usa GIN especificamente para columnas JSONB donde se consulta dentro de la estructura del documento, o para arrays donde se busca contencion de elementos.
3. Se usa GiST cuando se trabaja con datos geoespaciales (PostGIS) o rangos con necesidad de detectar solapamiento (por ejemplo, evitar reservas de horario que se superpongan).
4. Se evita crear indices Hash salvo un caso muy especifico de igualdad pura, dado que B-tree cubre ese caso igual de bien ademas de soportar rangos.
5. Se conecta esto con las columnas JSONB del capitulo de Tipos de Datos Avanzados, donde GIN es el indice natural para hacerlas consultables eficientemente.

## Criterio junior

Un junior debe poder crear un indice B-tree basico en una columna usada frecuentemente en `WHERE`.

## Criterio mid

Un mid debe elegir GIN para columnas JSONB o arrays que se consultan por contenido interno, distinguiendolo de cuando B-tree simple alcanza.

## Criterio senior

Un senior debe diseñar la estrategia de indexacion completa de un esquema, eligiendo el tipo correcto de indice segun el patron de consulta real de cada columna, y revisando en code review indices innecesarios o mal elegidos.

## Errores comunes

- Crear un indice GIN en una columna simple de texto o numero donde B-tree ya seria suficiente y mas liviano.
- No indexar columnas JSONB consultadas frecuentemente por su contenido interno, forzando a Postgres a escanear cada documento completo.
- Usar indices Hash sin necesidad real, cuando B-tree cubre el mismo caso de igualdad ademas de soportar mas operaciones.
- No conocer que tipo de indice corresponde a un tipo de dato especifico (JSONB, arrays, rangos), dejando esas columnas sin indexar apropiadamente.

## Practica

- [ ] Crea un indice B-tree en una columna de filtro frecuente y verifica su uso con `EXPLAIN` (ver siguiente tema).
- [ ] Crea una columna JSONB, indexala con GIN, y consulta dentro de su contenido.
- [ ] Crea una columna array, indexala con GIN, y consulta contencion de un elemento.
- [ ] Investiga un caso de uso real donde GiST seria apropiado (rangos o datos geoespaciales).
- [ ] Documenta una tabla de referencia propia: que tipo de indice corresponde a que tipo de dato/consulta.

## Preguntas de repaso

- Por que B-tree es el tipo de indice por defecto y cubre la mayoria de casos?
- Cuando elegirias GIN sobre B-tree para una columna?
- Que tipo de columna se beneficia especificamente de un indice GiST?
- Que limitacion tiene un indice Hash frente a B-tree?
