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

# 04 - Columnstore Indexes

## Que vas a aprender

En este capitulo vas a aprender **Columnstore Indexes**, una estructura de almacenamiento columnar (en vez de por filas) diseñada especificamente para cargas analiticas (OLAP), complementando todo lo ya estudiado sobre indices B-tree tradicionales orientados a OLTP.

## Explicacion

Un indice tradicional (rowstore, ya estudiado en el capitulo de indices) almacena los datos **fila por fila**, eficiente para operaciones OLTP que leen/escriben registros completos individuales. Un **Columnstore Index** almacena los datos **columna por columna**, con compresion muy agresiva (columnas del mismo tipo de dato comprimen mucho mejor juntas que filas heterogeneas), diseñado especificamente para queries analiticas que agregan (`SUM`, `AVG`, `COUNT`) sobre millones de filas pero tocan solo unas pocas columnas de cada una, exactamente el patron de acceso tipico de reportes y dashboards.

Existen dos variantes: **Clustered Columnstore Index** (reemplaza el almacenamiento completo de la tabla, apropiado para tablas de hechos de un data warehouse dedicado) y **Nonclustered Columnstore Index** (coexiste junto a un indice rowstore tradicional sobre la misma tabla, permitiendo que la misma tabla sirva eficientemente tanto queries OLTP transaccionales como queries analiticas ocasionales, un escenario **HTAP** - Hybrid Transactional/Analytical Processing). Postgres no tiene un equivalente nativo directo; el ecosistema resuelve este mismo problema con extensiones especializadas o herramientas externas de data warehousing.

## Modelo mental

Pensa un indice rowstore tradicional como un archivador donde cada carpeta contiene el expediente completo de una persona (todos sus datos juntos, eficiente para buscar "todo sobre Juan"). Un Columnstore Index es como reorganizar la informacion en registros separados por categoria (todas las edades juntas, todos los salarios juntos), ineficiente si necesitas el expediente completo de una persona, pero muy eficiente si necesitas calcular el promedio de todas las edades sin importarte nada mas de cada persona.

## Ejemplo base

```sql
-- Nonclustered Columnstore Index: coexiste con el indice rowstore tradicional
CREATE NONCLUSTERED COLUMNSTORE INDEX IX_Ventas_Columnstore
ON Ventas (ProductoId, Fecha, Monto, CantidadUnidades);

-- Query analitica que se beneficia enormemente del Columnstore Index
SELECT ProductoId, SUM(Monto) AS TotalVentas
FROM Ventas
WHERE Fecha >= '2026-01-01'
GROUP BY ProductoId;

-- Clustered Columnstore Index: reemplaza el almacenamiento completo (data warehouse)
CREATE CLUSTERED COLUMNSTORE INDEX IX_HechosVentas
ON HechosVentas;
```

## Como se usa en un proyecto real

1. Se agrega un Nonclustered Columnstore Index sobre una tabla transaccional (como una tabla de inscripciones o progreso de estudiantes en StaffTrainingPlatform) cuando se necesitan reportes analiticos frecuentes sin migrar los datos a un data warehouse separado.
2. Se usa Clustered Columnstore Index en tablas de hechos dentro de un data warehouse dedicado, donde la carga es predominantemente analitica y de solo lectura/carga masiva.
3. Se mide el impacto real en queries de reportes (agregaciones sobre millones de filas) antes y despues de agregar el Columnstore Index, confirmando el beneficio con evidencia.
4. Se entiende que Columnstore Index no ayuda (e incluso puede perjudicar) queries OLTP tipicas que leen/escriben filas individuales completas, reservandolo especificamente para el patron analitico.
5. Se documenta esta decision como parte de una estrategia HTAP consciente, no como una optimizacion generica aplicada sin entender el patron de acceso real.

## Criterio junior

Un junior debe entender la diferencia conceptual entre almacenamiento rowstore y columnar.

## Criterio mid

Un mid debe agregar un Nonclustered Columnstore Index apropiado para acelerar reportes analiticos sobre una tabla transaccional.

## Criterio senior

Un senior debe diseñar una estrategia HTAP consciente, decidiendo cuando Columnstore Index resuelve el problema versus cuando se justifica un data warehouse separado.

## Errores comunes

- Aplicar Columnstore Index sobre una tabla con carga predominantemente OLTP, sin beneficio real y con overhead de mantenimiento adicional.
- No medir el impacto real antes/despues, asumiendo el beneficio sin confirmarlo con evidencia.
- Confundir Clustered y Nonclustered Columnstore Index, aplicando el tipo equivocado para el escenario (transaccional vs data warehouse dedicado).
- Ignorar Columnstore Index como alternativa viable cuando se esta considerando migrar a un data warehouse separado innecesariamente pronto.

## Practica

- [ ] Crea un Nonclustered Columnstore Index sobre una tabla con datos de prueba de volumen considerable.
- [ ] Compara el tiempo de ejecucion de una query analitica de agregacion antes y despues del Columnstore Index.
- [ ] Investiga cuando conviene un Clustered Columnstore Index para una tabla de hechos dedicada.
- [ ] Identifica una tabla real (o hipotetica) de StaffTrainingPlatform que se beneficiaria de un escenario HTAP con Columnstore Index.
- [ ] Investiga como Postgres resuelve el mismo problema analitico sin un equivalente nativo directo.

## Preguntas de repaso

- Que diferencia hay entre almacenamiento rowstore y columnar, y por que el columnar favorece queries analiticas?
- Que diferencia hay entre Clustered y Nonclustered Columnstore Index?
- Que es un escenario HTAP, y como Nonclustered Columnstore Index lo habilita?
- Por que Columnstore Index no beneficia (e incluso puede perjudicar) queries OLTP tipicas?
