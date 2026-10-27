---
tags:
  - Postulaciones
  - SQLServer
  - PostgreSQL
  - NoSQL
Creado: 2026-08-30
Relacionado:
  - SQL SERVER
  - POSTGRESQL
  - RUTA FULL STACK ANGULAR NET AZURE
---

# SQL Server, PostgreSQL y NoSQL

## Decisión para esta ruta

1. **SQL relacional es obligatorio.**
2. **SQL Server es la prioridad inmediata** por la integración habitual con .NET y Azure SQL.
3. **PostgreSQL es el segundo motor recomendado** porque amplía las vacantes posibles y demuestra que el conocimiento de datos no depende de un proveedor.
4. **MongoDB es opcional**, útil cuando el modelo es documental; no debe aprenderse como almacén genérico de logs.

## Núcleo transferible entre SQL Server y PostgreSQL

- Modelado, normalización y constraints.
- Joins, agregaciones, CTEs y window functions.
- Índices y planes de ejecución.
- Transacciones, aislamiento, locks y deadlocks.
- Migraciones seguras.
- Seguridad, backups y connection pooling a nivel desarrollador.

Repasar primero: [[SQL Server con NET y EF Core]], [[01 - Indices Clustered y Non-Clustered]], [[01 - Tipos de Indices B-tree Hash GIN GiST]], [[01 - ACID en SQL Server]] y [[01 - ACID en Postgres]].

> [!warning]
> En el vault existen algunos títulos repetidos entre SQL Server y PostgreSQL. Usa los enlaces específicos anteriores o entra desde el índice [[SQL SERVER]] o [[POSTGRESQL]].

## Plan PostgreSQL mínimo viable — 10 horas

1. Instalar o levantar PostgreSQL con Docker.
2. Crear esquema con PK, FK, unique y checks.
3. Ejecutar joins, CTEs y transacciones.
4. Conectar una API .NET mediante el provider de PostgreSQL para EF Core.
5. Crear migrations separadas para ese proveedor.
6. Usar `EXPLAIN ANALYZE` y corregir una consulta.
7. Probar concurrencia y rollback.

Ruta: [[Fundamentos de PostgreSQL]], [[SQL Fundamentals en Postgres]], [[02 - EXPLAIN ANALYZE]], [[01 - Postgres en Docker]].

## Cuándo sí aprender MongoDB

- El puesto lo solicita repetidamente.
- Se almacenan documentos con estructura variable.
- El acceso principal recupera agregados completos por clave o filtros documentales.
- El proyecto necesita escalado y distribución propios de ese modelo.

## Cuándo no elegir MongoDB

- Relaciones y consistencia transaccional dominan el problema.
- Solo quieres decir que conoces NoSQL.
- Quieres guardar logs sin diseñar búsqueda, retención, alertas y correlación.
- El esquema cambia porque todavía no se ha modelado bien el dominio.

En Azure, una necesidad documental puede resolverse con Cosmos DB o una opción compatible con MongoDB, pero la decisión debe partir del patrón de acceso. La guía oficial recomienda elegir el almacén por modelo y requisitos, no por moda: [selección de almacén de datos en Azure](https://learn.microsoft.com/azure/architecture/guide/technology-choices/data-stores-getting-started).

## Qué usar para logs

- Emisión: `ILogger` y logs estructurados.
- Estandarización y correlación: OpenTelemetry.
- Backend Azure: Application Insights + Log Analytics.
- Alternativa local de portafolio: Seq o una pila Grafana/Loki.
- Elastic/OpenSearch: aprender si las vacantes lo piden o existe una necesidad fuerte de búsqueda.

## Criterio de entrevista

Debes poder comparar SQL Server, PostgreSQL y una base documental en términos de integridad, consultas, consistencia, escalado, operación y costo. No necesitas administrarlos todos a nivel DBA.
