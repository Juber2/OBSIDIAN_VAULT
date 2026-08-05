---
tags:
  - SQLServer
  - Proyectos
  - SQLServer-Proyectos
Creado: 2026-07-29
Relacionado:
  - Proyectos
  - SQL SERVER
---

# 02 - Proyecto Optimizacion de Queries

## Que vas a aprender

En este proyecto vas a aplicar los capitulos de Indices y Optimizacion, T-SQL Avanzado, y Administracion, tomando el esquema del proyecto anterior (poblado con datos de volumen considerable) y optimizando queries deliberadamente lentas usando Execution Plans, indices apropiados, y reescritura de T-SQL.

## Explicacion

Este proyecto simula el trabajo real de un desarrollador o DBA enfrentando un sistema existente con problemas de performance: partis de un esquema ya poblado con decenas de miles (o mas) de filas de datos sinteticos, identificas queries lentas mediante Query Store o DMVs (`sys.dm_exec_query_stats`), analizas su Execution Plan para entender la causa raiz (table scan innecesario, indice faltante, Parameter Sniffing, uso de cursor donde un JOIN bastaria), y aplicas la correccion apropiada, midiendo el impacto real antes y despues de cada cambio.

Es importante que cada optimizacion este respaldada por evidencia medida, no por intuicion: el ejercicio central de este proyecto es practicar el flujo completo de "medir, diagnosticar, corregir, volver a medir" ya insistido a lo largo de todo el libro, no memorizar soluciones genericas sin entender por que funcionan en cada caso especifico.

## Modelo mental

Pensa este proyecto como trabajar de detective de performance: no asumis quien es el culpable (que query es lenta) sin evidencia, investigas con las herramientas correctas (Query Store, DMVs, Execution Plans), y una vez que confirmas la causa, aplicas la correccion especifica para ese caso, verificando despues que el problema realmente se resolvio.

## Ejemplo base

```sql
-- Poblar la tabla Inscripciones con datos de volumen considerable
INSERT INTO Inscripciones (EstudianteId, CursoId, FechaInscripcion)
SELECT TOP 100000
    ABS(CHECKSUM(NEWID())) % 1000 + 1,
    ABS(CHECKSUM(NEWID())) % 50 + 1,
    DATEADD(DAY, -ABS(CHECKSUM(NEWID())) % 365, SYSUTCDATETIME())
FROM sys.all_objects a CROSS JOIN sys.all_objects b;

-- Identificar queries lentas con Query Store
SELECT qt.query_sql_text, rs.avg_duration
FROM sys.query_store_query q
JOIN sys.query_store_query_text qt ON q.query_text_id = qt.query_text_id
JOIN sys.query_store_plan p ON q.query_id = p.query_id
JOIN sys.query_store_runtime_stats rs ON p.plan_id = rs.plan_id
ORDER BY rs.avg_duration DESC;

-- Analizar el Execution Plan de la query mas lenta identificada
SET STATISTICS IO ON;
-- ejecutar la query sospechosa aca
```

## Como se usa en un proyecto real

1. Se pueblan las tablas del proyecto anterior con datos sinteticos de volumen considerable, simulando condiciones reales de produccion.
2. Se habilita Query Store y se identifican las queries con mayor tiempo de ejecucion promedio, usando evidencia objetiva en vez de sospechas.
3. Se analiza el Execution Plan de cada query lenta identificada, buscando table scans innecesarios, indices faltantes, o operadores costosos.
4. Se aplica la correccion apropiada (nuevo indice, reescritura de la query, eliminacion de un cursor) y se mide el impacto real comparando el estado antes/despues.
5. Se documenta cada optimizacion con la evidencia que la justifico, replicando el rigor esperado en un ambiente de produccion real.

## Criterio de exito

- Identificaste al menos tres queries lentas usando Query Store o DMVs, con evidencia objetiva de su costo.
- Analizaste el Execution Plan de cada una, identificando la causa raiz especifica del problema.
- Aplicaste una correccion medible para cada caso, con el impacto confirmado antes/despues.
- Documentaste el proceso completo como si fuera un reporte de optimizacion real para el equipo.

## Errores comunes

- Aplicar indices o cambios sin medir primero, basandose en intuicion en vez de evidencia de Query Store/DMVs.
- Crear indices redundantes o innecesarios que no corresponden a ningun problema real identificado.
- No verificar el impacto real despues de aplicar una correccion, asumiendo que funciono sin confirmarlo.
- Optimizar queries que en realidad no son las mas costosas, perdiendo tiempo en problemas de bajo impacto real.

## Practica

- [ ] Puebla el esquema del proyecto anterior con datos sinteticos de volumen considerable.
- [ ] Identifica las tres queries mas lentas usando Query Store.
- [ ] Analiza el Execution Plan de cada una y documenta la causa raiz encontrada.
- [ ] Aplica la correccion apropiada para cada caso y mide el impacto real.
- [ ] Escribe un reporte breve documentando el proceso completo, como si fuera para tu equipo.

## Preguntas de repaso

- Que herramienta usaste para identificar las queries mas lentas, y por que?
- Que causa raiz especifica encontraste en cada query analizada?
- Que correccion aplicaste, y como confirmaste su impacto real?
- Que harias diferente si este mismo ejercicio lo repitieras en un sistema de produccion real?
