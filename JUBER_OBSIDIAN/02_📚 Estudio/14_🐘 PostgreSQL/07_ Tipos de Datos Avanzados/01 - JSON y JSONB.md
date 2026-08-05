---
tags:
  - PostgreSQL
  - Tipos-de-Datos
  - Postgres-Avanzado
Creado: 2026-07-29
Relacionado:
  - Tipos de Datos Avanzados
  - POSTGRESQL
---

# 01 - JSON y JSONB

## Que vas a aprender

En este capitulo vas a dominar los tipos `json` y `jsonb` de Postgres, entendiendo la diferencia critica entre ambos, operadores de consulta dentro de documentos JSON, e indexacion con GIN (ya visto en el capitulo de Indices).

## Explicacion

Postgres ofrece dos tipos para datos JSON: **`json`** almacena el texto tal cual se ingreso (preservando formato, orden de claves, espacios), sin ningun procesamiento adicional al guardar, pero re-parsea el texto completo en cada consulta. **`jsonb`** ("JSON binario") almacena una representacion binaria ya procesada y optimizada, mas rapida de consultar (no necesita re-parsear) y **indexable con GIN**, a costa de perder el formato de texto original exacto (orden de claves no se preserva, espacios se eliminan). En la practica, `jsonb` es casi siempre la eleccion correcta salvo que se necesite preservar el texto original exacto.

Los operadores clave son `->` (extrae un valor como JSON), `->>` (extrae un valor como texto), y `@>` (contiene, util para buscar documentos que incluyen ciertos pares clave-valor). Combinado con un indice GIN, `jsonb` permite consultas eficientes dentro de documentos semi-estructurados sin necesitar definir un esquema rigido de columnas, util para datos con forma variable (metadata de producto, configuracion de usuario, respuestas de formularios dinamicos).

## Modelo mental

Pensa `json` como guardar una carta manuscrita exactamente como llego, preservando cada detalle de formato original, pero teniendo que releerla completa cada vez que necesitas buscar algo en ella. `jsonb` es como haber transcrito esa carta a un formato digital indexado y buscable: perdiste la caligrafia original exacta, pero ahora podes encontrar cualquier informacion dentro de ella instantaneamente.

## Ejemplo base

```sql
CREATE TABLE productos (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre text NOT NULL,
    metadata jsonb
);

INSERT INTO productos (nombre, metadata) VALUES
    ('Camisa', '{"color": "azul", "talles": ["S", "M", "L"], "en_oferta": true}');

SELECT metadata->>'color' FROM productos;               -- extrae como texto: "azul"
SELECT metadata->'talles' FROM productos;                -- extrae como JSON: ["S", "M", "L"]

SELECT * FROM productos WHERE metadata @> '{"en_oferta": true}';   -- contiene ese par clave-valor
SELECT * FROM productos WHERE metadata->>'color' = 'azul';

CREATE INDEX idx_productos_metadata ON productos USING gin (metadata);
```

## Como se usa en un proyecto real

1. Se usa `jsonb` como default sobre `json` en cualquier caso nuevo, salvo la rara necesidad de preservar el texto JSON original exacto.
2. Se indexa con GIN cualquier columna `jsonb` consultada frecuentemente por su contenido interno, evitando escaneos completos de la tabla.
3. Se usa `jsonb` para datos genuinamente semi-estructurados (metadata variable, configuracion flexible), sin abusar de el para modelar datos que en realidad son relacionales y se beneficiarian de columnas y tablas normales.
4. Se conecta esto con TypeORM del libro de NestJS, donde una columna `jsonb` mapea directamente a un tipo de columna `jsonb` en el decorador de entidad.
5. Se documenta la estructura esperada de un campo `jsonb` (aunque no este forzada por el esquema), como contrato implicito para el equipo.

## Criterio junior

Un junior debe poder crear una columna `jsonb`, insertar datos, y extraer valores con `->`/`->>`.

## Criterio mid

Un mid debe usar `@>` para consultas de contencion, e indexar columnas `jsonb` consultadas frecuentemente con GIN.

## Criterio senior

Un senior debe decidir con criterio cuando `jsonb` es apropiado (datos genuinamente semi-estructurados) versus cuando el diseño relacional normal seria mejor, evitando el abuso de JSONB como "escape hatch" de mal modelado.

## Errores comunes

- Usar `json` en vez de `jsonb` sin una razon real para preservar el formato de texto original.
- No indexar con GIN una columna `jsonb` consultada frecuentemente, generando escaneos completos costosos de la tabla.
- Abusar de `jsonb` para modelar datos que en realidad son relacionales (por ejemplo, una lista de items que deberia ser una tabla separada), perdiendo los beneficios de constraints e integridad referencial.
- No documentar la estructura esperada de un campo `jsonb`, generando inconsistencia de formato entre distintas filas con el tiempo.

## Practica

- [ ] Crea una tabla con una columna `jsonb` e inserta datos con estructura anidada.
- [ ] Extrae valores con `->` y `->>`, comparando el tipo de resultado de cada uno.
- [ ] Consulta documentos que contienen un par clave-valor especifico con `@>`.
- [ ] Indexa la columna `jsonb` con GIN y verifica con `EXPLAIN ANALYZE` que se usa.
- [ ] Identifica un caso de tu proyecto donde `jsonb` seria apropiado, y otro donde en realidad convendria una tabla relacional normal.

## Preguntas de repaso

- Que diferencia hay entre `json` y `jsonb`, y por que `jsonb` es casi siempre la eleccion correcta?
- Que diferencia hay entre los operadores `->` y `->>`?
- Para que sirve el operador `@>` sobre una columna `jsonb`?
- Cuando el uso de `jsonb` seria un sintoma de mal modelado relacional?
