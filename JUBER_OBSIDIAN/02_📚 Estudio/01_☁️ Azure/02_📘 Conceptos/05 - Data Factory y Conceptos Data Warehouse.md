---
tags:
  - Azure
  - Data
Creado: 2026-08-12
Relacionado:
  - Azure
---

# Data Factory y Conceptos de Data Warehouse (deseables del puesto)

## Azure Data Factory: orquestación de pipelines de datos (ETL/ELT)

Servicio PaaS para mover y transformar datos entre sistemas — típicamente de bases de datos operacionales (OLTP) hacia un Data Warehouse (para análisis), sin escribir infraestructura de procesamiento desde cero.

```
Origen (SQL Server, APIs, archivos) → Data Factory (pipeline) → Destino (Data Warehouse, Data Lake)
```

Componentes clave:
- **Pipeline**: el flujo completo (secuencia de actividades).
- **Activity**: un paso individual (copiar datos, ejecutar un stored procedure, transformar con Data Flow).
- **Linked Service**: la conexión a un sistema origen/destino (connection string a una BD, credenciales a un storage).
- **Dataset**: la representación de los datos dentro de un Linked Service (una tabla, un archivo).
- **Trigger**: qué dispara el pipeline (schedule, evento, manual).

Para un backend Java, lo relevante en entrevista suele ser el **rol conceptual**: entender que Data Factory es la pieza que mueve datos de tu base operacional (la que tu API escribe) hacia un entorno analítico, no que vayas a programarlo vos mismo como developer backend.

## OLTP vs OLAP

| | OLTP (Online Transaction Processing) | OLAP (Online Analytical Processing) |
|---|---|---|
| Propósito | operaciones del día a día (crear pedido, actualizar stock) | análisis histórico, reportes, BI |
| Modelo de datos | normalizado (evita redundancia) | desnormalizado (optimizado para lectura/agregación) |
| Volumen por operación | filas individuales | grandes volúmenes agregados |
| Ejemplo | la base de datos de tu API Spring/Quarkus | un Data Warehouse consultado desde Power BI |

Una API Java típica (con JPA/Hibernate) trabaja siempre sobre un modelo **OLTP** — el Data Warehouse es un sistema separado, alimentado desde ahí vía ETL/ELT, no la misma base de datos.

## DataMart

Un **DataMart** es un subconjunto de un Data Warehouse, enfocado en un área de negocio específica (ej. "DataMart de Ventas", "DataMart de RRHH"). Se usa para que cada área consulte solo lo que le interesa, con mejor performance que consultar el Data Warehouse completo.

```
Data Warehouse (todo el negocio)
   ├── DataMart Ventas
   ├── DataMart Finanzas
   └── DataMart Operaciones
```

## Data Vault

Metodología de modelado para Data Warehouses, alternativa al modelo dimensional clásico (estrella/copo de nieve), diseñada para ser más resistente a cambios en el negocio y auditable históricamente. Se compone de tres tipos de entidades:

| Entidad | Qué guarda |
|---|---|
| **Hub** | la clave de negocio (ej. ID de cliente), sin atributos descriptivos |
| **Satellite** | los atributos descriptivos y su historial de cambios (ej. nombre, dirección, con fecha de vigencia) |
| **Link** | las relaciones entre Hubs (ej. relación Cliente-Pedido) |

Ventaja principal: agregar una fuente de datos nueva no rompe el modelo existente (a diferencia del modelo dimensional, más rígido) — trade-off es mayor complejidad de queries.

## Por qué esto le importa a un backend developer

No se espera que un Senior Java Developer diseñe Data Warehouses — pero sí que entienda:
1. Que su API (OLTP) es una **fuente** para pipelines de Data Factory, y diseñar el modelo pensando en que otros sistemas van a leer/extraer de ahí (ej. no borrar físicamente registros importantes para analítica, usar soft-delete, mantener timestamps de auditoría).
2. La diferencia OLTP/OLAP para no proponer "hacer los reportes pesados directo contra la base transaccional" — error común que degrada el rendimiento de la aplicación en producción.

## Errores comunes (conceptuales)

- Confundir Data Factory con una herramienta de transformación en tiempo real — es orquestación batch/programada, no streaming (para eso Azure tiene Stream Analytics/Event Hubs).
- Ejecutar reportes analíticos pesados directo contra la base OLTP de producción, degradando la app real.
- No diferenciar DataMart de Data Warehouse en una conversación técnica (son términos relacionados pero no intercambiables).

## Checklist
- [ ] Puedo explicar qué hace Azure Data Factory a alto nivel (orquestación ETL/ELT).
- [ ] Distingo claramente OLTP de OLAP con ejemplos.
- [ ] Sé qué es un DataMart y cómo se relaciona con un Data Warehouse.
- [ ] Entiendo la idea general de Data Vault (Hub/Satellite/Link) aunque no lo implemente yo mismo.
- [ ] Puedo argumentar por qué no correr analítica pesada contra la base transaccional de producción.
