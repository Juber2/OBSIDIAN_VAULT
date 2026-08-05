---
tags:
  - SQLServer
  - PerformanceAvanzada
  - SQLServer-Performance
Creado: 2026-07-29
Relacionado:
  - Performance Avanzada
  - SQL SERVER
---

# 01 - Particionamiento de Tablas

## Que vas a aprender

En este capitulo vas a aprender **Table Partitioning** de SQL Server: dividir una tabla grande en particiones fisicas segun una columna (tipicamente una fecha), aplicando el mismo concepto ya dominado en profundidad en el capitulo de particionamiento de Postgres.

## Explicacion

El particionamiento en SQL Server se configura con una **Partition Function** (define los rangos de valores, por ejemplo por año o mes de una columna fecha) y un **Partition Scheme** (mapea cada rango de la funcion hacia un filegroup fisico especifico), una arquitectura mas explicita en dos pasos comparada con `PARTITION BY RANGE` directo de Postgres, pero resolviendo exactamente el mismo problema: consultas que filtran por la columna de particion solo escanean las particiones relevantes (**partition elimination**, equivalente a partition pruning de Postgres), y el mantenimiento (purgar datos antiguos) se vuelve una operacion de metadatos casi instantanea (`SWITCH PARTITION`) en vez de un `DELETE` masivo costoso.

Al igual que en Postgres, particionar prematuramente sobre tablas pequeñas añade complejidad sin beneficio real; el particionamiento se justifica cuando la tabla crece a decenas de millones de filas y las queries tipicas ya filtran naturalmente por la columna candidata a particion (fecha de creacion, tenant_id, etc.).

## Modelo mental

Pensa el particionamiento de SQL Server exactamente igual que en Postgres: en vez de un archivador gigante con todos los documentos de los ultimos diez años mezclados, tenes un cajon separado por año. Buscar documentos de un año especifico solo revisa ese cajon (partition elimination), y tirar los documentos de hace diez años es sacar el cajon completo (`SWITCH PARTITION`) en vez de revisar documento por documento para decidir cual tirar.

## Ejemplo base

```sql
-- Partition Function: define los rangos
CREATE PARTITION FUNCTION PF_PorAño (DATE)
    AS RANGE RIGHT FOR VALUES ('2024-01-01', '2025-01-01', '2026-01-01');

-- Partition Scheme: mapea rangos a filegroups
CREATE PARTITION SCHEME PS_PorAño
    AS PARTITION PF_PorAño ALL TO ([PRIMARY]);

-- Tabla particionada
CREATE TABLE Eventos (
    Id INT IDENTITY,
    Fecha DATE NOT NULL,
    Descripcion NVARCHAR(200)
) ON PS_PorAño(Fecha);

-- SWITCH PARTITION: purgar datos antiguos casi instantaneamente
ALTER TABLE Eventos SWITCH PARTITION 1 TO EventosArchivo PARTITION 1;
```

## Como se usa en un proyecto real

1. Se particiona una tabla de logs, auditoria, o eventos de alto volumen (potencialmente aplicable a tablas de progreso de estudiantes en StaffTrainingPlatform si crecen a gran escala) por fecha, cuando las queries tipicas ya filtran por rango de fechas.
2. Se usa `SWITCH PARTITION` para purgar datos antiguos de forma casi instantanea, evitando el costo de un `DELETE` masivo que generaria un log de transacciones enorme.
3. Se verifica que las queries realmente aprovechan partition elimination revisando el Execution Plan, el mismo habito ya establecido en el capitulo de indices.
4. Se conecta esto directamente con el particionamiento ya dominado en Postgres, aplicando el mismo criterio de cuando particionar vale la pena.
5. Se evita particionar tablas pequeñas o donde las queries no filtran naturalmente por la columna candidata, evitando complejidad sin beneficio medible.

## Criterio junior

Un junior debe entender el proposito del particionamiento y poder crear una tabla particionada basica siguiendo un ejemplo.

## Criterio mid

Un mid debe diseñar una Partition Function apropiada para un caso de uso real y verificar partition elimination en el Execution Plan.

## Criterio senior

Un senior debe decidir cuando el particionamiento realmente se justifica en produccion, y diseñar la estrategia de mantenimiento (purga con `SWITCH PARTITION`) asociada.

## Errores comunes

- Particionar una tabla pequeña o donde las queries no filtran por la columna de particion, agregando complejidad sin beneficio real.
- No verificar que las queries realmente logran partition elimination, asumiendo el beneficio sin confirmarlo en el Execution Plan.
- Usar `DELETE` masivo en vez de `SWITCH PARTITION` para purgar datos antiguos, generando un log de transacciones innecesariamente grande.
- Diseñar rangos de particion que no se alinean con el patron real de crecimiento de los datos, resultando en particiones desbalanceadas.

## Practica

- [ ] Crea una tabla particionada por fecha con Partition Function y Partition Scheme.
- [ ] Inserta datos de prueba en distintas particiones y verifica partition elimination en el Execution Plan.
- [ ] Practica `SWITCH PARTITION` para mover datos de una particion hacia una tabla de archivo.
- [ ] Compara esta sintaxis de dos pasos con `PARTITION BY RANGE` directo de Postgres.
- [ ] Identifica una tabla real (o hipotetica) de StaffTrainingPlatform que se beneficiaria de particionamiento.

## Preguntas de repaso

- Que diferencia hay entre una Partition Function y un Partition Scheme?
- Que es partition elimination, y como lo verificarias en un Execution Plan?
- Por que `SWITCH PARTITION` es preferible a un `DELETE` masivo para purgar datos antiguos?
- Como se compara esta sintaxis con el particionamiento ya estudiado en Postgres?
