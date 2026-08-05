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

# 04 - Full Text Search

## Que vas a aprender

En este capitulo vas a aprender **Full Text Search** nativo de Postgres: los tipos `tsvector`/`tsquery`, la diferencia con una busqueda `ILIKE` simple, y como construir busqueda de texto relevante y ordenada por ranking sin necesitar un motor de busqueda externo como Elasticsearch para casos moderados.

## Explicacion

Una busqueda con `ILIKE '%palabra%'` (ya vista en el capitulo de SELECT) es simple pero limitada: no entiende variaciones de palabra (singular/plural, conjugaciones), no puede ordenar resultados por relevancia, y no puede usar indices eficientemente para busquedas de subcadena en medio del texto. **Full Text Search** de Postgres convierte texto en un **`tsvector`** (una representacion normalizada, con las palabras reducidas a su raiz lexica -stemming- y stop words como "el", "la", "de" removidas), y una consulta de busqueda en un **`tsquery`**, comparables con el operador `@@`.

El resultado se puede ordenar por **relevancia** con `ts_rank()`, dando resultados donde coincidencias mas fuertes aparecen primero, similar a como funcionaria un motor de busqueda real. Esta funcionalidad, indexada con GIN, cubre las necesidades de busqueda de texto de la mayoria de aplicaciones sin necesitar la complejidad operativa de un motor de busqueda externo dedicado.

## Modelo mental

Pensa `ILIKE` como buscar literalmente una secuencia exacta de letras en un texto, sin entender el idioma. Full Text Search es como tener a alguien que realmente entiende el idioma: sabe que "corriendo" y "correr" son la misma idea (stemming), ignora palabras sin significado real como "el" o "de" (stop words), y puede decirte que tan relevante es cada resultado encontrado, no solo si coincide o no.

## Ejemplo base

```sql
ALTER TABLE productos ADD COLUMN busqueda tsvector
    GENERATED ALWAYS AS (to_tsvector('spanish', nombre || ' ' || descripcion)) STORED;

CREATE INDEX idx_productos_busqueda ON productos USING gin (busqueda);

SELECT nombre, ts_rank(busqueda, query) AS relevancia
FROM productos, to_tsquery('spanish', 'camisa & azul') AS query
WHERE busqueda @@ query
ORDER BY relevancia DESC;

-- to_tsquery entiende variaciones de palabra gracias al stemming
SELECT to_tsvector('spanish', 'Corriendo por el parque') @@ to_tsquery('spanish', 'correr');
-- true, aunque el texto original dice "corriendo" y la busqueda es "correr"
```

## Como se usa en un proyecto real

1. Se usa Full Text Search para funcionalidades de busqueda de texto real dentro de la aplicacion (buscar productos, articulos, contenido), en vez de `ILIKE` simple cuando se necesita relevancia y tolerancia a variaciones de palabra.
2. Se genera la columna `tsvector` como columna calculada (`GENERATED ALWAYS AS ... STORED`) para mantenerla siempre sincronizada automaticamente sin necesitar un trigger manual.
3. Se indexa la columna `tsvector` con GIN, indispensable para performance aceptable en tablas grandes.
4. Se ordena por `ts_rank()` cuando la relevancia importa para la experiencia de busqueda del usuario, no solo la coincidencia binaria.
5. Se evalua migrar a un motor de busqueda externo (Elasticsearch) solo cuando las necesidades superan claramente lo que Full Text Search de Postgres puede ofrecer (busqueda facetada compleja, volumen extremadamente alto), evitando esa complejidad operativa adicional sin necesidad real.

## Criterio junior

Un junior debe poder crear una columna `tsvector` basica y consultarla con `@@` y `to_tsquery`.

## Criterio mid

Un mid debe generar columnas `tsvector` automaticamente con `GENERATED ALWAYS AS ... STORED`, indexarlas con GIN, y ordenar resultados por `ts_rank()`.

## Criterio senior

Un senior debe decidir cuando Full Text Search de Postgres es suficiente versus cuando un motor de busqueda externo se justifica, evitando la complejidad operativa adicional sin necesidad real comprobada.

## Errores comunes

- Usar `ILIKE` para funcionalidades de busqueda que se beneficiarian claramente de relevancia y tolerancia a variaciones de palabra que Full Text Search ofrece.
- No indexar la columna `tsvector` con GIN, perdiendo el beneficio de performance que justifica usar esta feature.
- Actualizar manualmente una columna `tsvector` en vez de usar `GENERATED ALWAYS AS ... STORED`, arriesgando que quede desincronizada del contenido real.
- Migrar prematuramente a un motor de busqueda externo cuando Full Text Search de Postgres ya cubriria las necesidades reales del proyecto.

## Practica

- [ ] Crea una columna `tsvector` generada automaticamente a partir de columnas de texto existentes.
- [ ] Indexa esa columna con GIN.
- [ ] Realiza una busqueda con `to_tsquery` que aproveche el stemming (buscando una variacion de palabra distinta a la del texto original).
- [ ] Ordena resultados de busqueda por `ts_rank()`.
- [ ] Evalua si un proyecto real tuyo necesitaria Full Text Search de Postgres o justificaria un motor externo.

## Preguntas de repaso

- Que limitacion tiene `ILIKE` que Full Text Search resuelve?
- Que hace el stemming en el contexto de Full Text Search?
- Por que generar la columna `tsvector` con `GENERATED ALWAYS AS ... STORED` es mejor que actualizarla manualmente?
- Cuando se justificaria migrar de Full Text Search de Postgres a un motor de busqueda externo?
