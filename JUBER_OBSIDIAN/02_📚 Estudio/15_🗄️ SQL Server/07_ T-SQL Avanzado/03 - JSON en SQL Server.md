---
tags:
  - SQLServer
  - TSQL-Avanzado
  - SQLServer-Avanzado
Creado: 2026-07-29
Relacionado:
  - T-SQL Avanzado
  - SQL SERVER
---

# 03 - JSON en SQL Server

## Que vas a aprender

En este capitulo vas a aprender el soporte de JSON en SQL Server (`FOR JSON`, `OPENJSON`, `JSON_VALUE`), notablemente distinto al tipo `jsonb` nativo de Postgres ya estudiado en profundidad en ese libro: SQL Server almacena JSON como **texto** (`NVARCHAR(MAX)`), no como un tipo binario dedicado.

## Explicacion

A diferencia de Postgres, que tiene un tipo `jsonb` nativo con almacenamiento binario optimizado e indices GIN dedicados (ver ese libro), SQL Server maneja JSON como **texto plano** almacenado en columnas `NVARCHAR(MAX)`, con funciones especiales para trabajar con ese texto como si fuera JSON estructurado: `JSON_VALUE()` extrae un valor escalar, `JSON_QUERY()` extrae un objeto o array, `OPENJSON()` convierte un JSON en un conjunto de filas (similar en espiritu a `jsonb_each`/`unnest` de Postgres), y `FOR JSON AUTO/PATH` convierte el resultado de una consulta relacional **en** JSON, util para APIs.

Esta diferencia de implementacion (texto vs binario nativo) significa que las operaciones JSON en SQL Server son generalmente **menos eficientes** que en Postgres para volumenes grandes de consultas frecuentes sobre contenido JSON, siendo un factor real a considerar si un proyecto depende fuertemente de datos semi-estructurados.

## Modelo mental

Pensa el JSON de Postgres como un documento ya escaneado y indexado digitalmente, listo para busqueda instantanea de cualquier palabra dentro de el (JSONB binario con GIN). El JSON de SQL Server es como un documento de texto plano que, cada vez que necesitas buscar algo dentro de el, hay que releerlo completo interpretandolo como JSON en el momento (aunque SQL Server puede indexar expresiones calculadas sobre ese JSON para mitigar parcialmente este costo).

## Ejemplo base

```sql
CREATE TABLE Productos (
    Id BIGINT IDENTITY PRIMARY KEY,
    Metadata NVARCHAR(MAX)   -- JSON almacenado como texto
);

INSERT INTO Productos (Metadata) VALUES
    ('{"color": "azul", "talles": ["S", "M", "L"]}');

SELECT JSON_VALUE(Metadata, '$.color') FROM Productos;   -- extrae "azul"

SELECT * FROM Productos
WHERE JSON_VALUE(Metadata, '$.color') = 'azul';

-- OPENJSON: convertir JSON en filas
SELECT p.Id, valor.value AS Talle
FROM Productos p
CROSS APPLY OPENJSON(p.Metadata, '$.talles') AS valor;

-- FOR JSON: convertir resultado relacional en JSON (util para APIs)
SELECT Id, Nombre FROM Productos FOR JSON AUTO;
```

## Como se usa en un proyecto real

1. Se usa `JSON_VALUE()`/`JSON_QUERY()` para extraer datos de columnas JSON en T-SQL, entendiendo que internamente sigue siendo texto reinterpretado en cada consulta.
2. Se evalua un **indice calculado persistido** (`ALTER TABLE ... ADD ColorCalculado AS JSON_VALUE(Metadata, '$.color') PERSISTED`, con un indice normal sobre esa columna calculada) para mitigar la falta de indexacion nativa directa sobre contenido JSON.
3. Se usa `FOR JSON AUTO/PATH` para que SQL Server genere directamente la respuesta JSON de una API desde T-SQL, util en ciertos escenarios de integracion.
4. Se compara conscientemente esta limitacion con el soporte nativo de `jsonb` de Postgres al decidir que motor usar para un proyecto con fuerte dependencia de datos semi-estructurados.
5. Se conecta esto con Entity Framework Core, que tambien soporta mapear propiedades como JSON (`OwnsMany`/conversiones), abstrayendo parcialmente estas diferencias de implementacion.

## Criterio junior

Un junior debe poder extraer valores de una columna JSON con `JSON_VALUE()`.

## Criterio mid

Un mid debe usar `OPENJSON()` para convertir un array JSON en filas consultables, y `FOR JSON` para generar respuestas JSON desde SQL.

## Criterio senior

Un senior debe evaluar el costo de performance del soporte JSON de SQL Server (texto reinterpretado) frente al soporte nativo de Postgres, decidiendo con criterio si un proyecto con fuerte dependencia de JSON deberia considerar Postgres en su lugar.

## Errores comunes

- Asumir que el soporte JSON de SQL Server es tan eficiente como el `jsonb` nativo de Postgres, sin considerar que internamente sigue siendo texto reinterpretado.
- No usar columnas calculadas persistidas con indice para mitigar el costo de consultas JSON frecuentes.
- No comparar esta limitacion real al decidir entre Postgres y SQL Server para un proyecto con fuerte dependencia de datos semi-estructurados.
- No aprovechar `FOR JSON` cuando generar respuestas JSON directamente desde SQL simplificaria la capa de aplicacion.

## Practica

- [ ] Crea una columna con JSON almacenado como texto e inserta datos de ejemplo.
- [ ] Extrae valores con `JSON_VALUE()` y convierte un array con `OPENJSON()`.
- [ ] Crea una columna calculada persistida indexada para mejorar consultas JSON frecuentes.
- [ ] Genera una respuesta JSON desde una consulta relacional con `FOR JSON AUTO`.
- [ ] Compara explicitamente esta implementacion con el soporte nativo `jsonb` ya estudiado en Postgres.

## Preguntas de repaso

- Que diferencia fundamental de implementacion hay entre el JSON de SQL Server y el `jsonb` de Postgres?
- Como mitigarias el costo de consultas frecuentes sobre contenido JSON en SQL Server?
- Para que sirve `FOR JSON AUTO/PATH`?
- Que factor real considerarias al elegir entre SQL Server y Postgres para un proyecto con fuerte dependencia de datos JSON?
