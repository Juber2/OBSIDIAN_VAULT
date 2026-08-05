---
tags:
  - SQLServer
  - Performance
  - SQLServer-Indices
Creado: 2026-07-29
Relacionado:
  - Indices y Optimizacion de Queries
  - SQL SERVER
---

# 02 - Execution Plans

## Que vas a aprender

En este capitulo vas a aprender a leer **Execution Plans** de SQL Server, el equivalente exacto de `EXPLAIN ANALYZE` de Postgres (ya estudiado en profundidad en ese libro), con la ventaja de tener una **representacion visual grafica** nativa en SSMS/Azure Data Studio.

## Explicacion

Un Execution Plan muestra como SQL Server decide ejecutar una consulta, identico en proposito a `EXPLAIN ANALYZE`: se puede ver el plan **estimado** (antes de ejecutar, `Ctrl+L` en SSMS) o el **real** (despues de ejecutar, con estadisticas reales de filas y tiempo, `Ctrl+M` para incluirlo). Operaciones clave a identificar: un **Table Scan** o **Clustered Index Scan** (recorre toda la tabla, equivalente al Seq Scan de Postgres, costoso en tablas grandes) versus un **Index Seek** (busca directamente usando un indice, equivalente al Index Scan eficiente de Postgres).

La representacion **grafica** es una ventaja practica real de SQL Server sobre la salida de texto de `EXPLAIN` de Postgres: cada operacion se muestra como un icono con el **porcentaje de costo relativo** del total de la consulta directamente visible, facilitando identificar rapidamente donde se concentra el costo sin necesitar leer texto denso.

## Modelo mental

El modelo mental es identico al ya establecido con `EXPLAIN ANALYZE` en Postgres: un plan de ejecucion es el mapa de ruta que el motor decidio tomar, y un Scan completo es recorrer cada casa de una ciudad buscando una direccion, mientras que un Seek es ir directo a la cuadra correcta gracias a un indice. La diferencia practica es que SQL Server te muestra ese mapa como un dibujo visual con "peajes" marcados (porcentaje de costo), en vez de solo texto para interpretar.

## Ejemplo base

```sql
-- Habilitar estadisticas de ejecucion real (equivalente a EXPLAIN ANALYZE)
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

SELECT * FROM Pedidos WHERE ClienteId = 42 ORDER BY CreadoEn DESC;

-- Sin indice: Clustered Index Scan (recorre toda la tabla), alto costo relativo
-- Con indice en (ClienteId, CreadoEn): Index Seek, bajo costo relativo
```

```sql
-- Ver el plan de ejecucion estimado sin ejecutar (via T-SQL, equivalente a EXPLAIN sin ANALYZE)
SET SHOWPLAN_XML ON;
GO
SELECT * FROM Pedidos WHERE ClienteId = 42;
GO
SET SHOWPLAN_XML OFF;
```

## Como se usa en un proyecto real

1. Se usa el Execution Plan grafico en Azure Data Studio/SSMS como primer paso ante cualquier query lenta, aplicando el mismo proceso de diagnostico ya dominado con `EXPLAIN ANALYZE`.
2. Se busca especificamente **Table Scan**/**Clustered Index Scan** en tablas grandes como señal de que falta un indice apropiado.
3. Se aprovecha el porcentaje de costo relativo visible en el plan grafico para identificar rapidamente cual operacion especifica dentro de una consulta compleja es la mas costosa.
4. Se usa `SET STATISTICS IO/TIME ON` para obtener metricas textuales complementarias (lecturas logicas, tiempo de CPU) junto al plan grafico.
5. Se conecta esto directamente con el capitulo de EXPLAIN ANALYZE del libro de Postgres, transfiriendo el mismo proceso de diagnostico con la sintaxis y herramientas especificas de SQL Server.

## Criterio junior

Un junior debe poder generar un Execution Plan grafico basico e identificar si una consulta usa Scan o Seek.

## Criterio mid

Un mid debe leer un plan de ejecucion complejo con multiples operaciones, identificando el porcentaje de costo relativo de cada una.

## Criterio senior

Un senior debe usar Execution Plans sistematicamente en revisiones de performance, aplicando el mismo rigor de "medir antes de optimizar" ya establecido en el libro de Postgres.

## Errores comunes

- Agregar indices "a ciegas" sin haber revisado el Execution Plan primero para confirmar que realmente falta uno en ese punto.
- No distinguir entre plan estimado y plan real, perdiendo la oportunidad de comparar estimaciones del optimizador contra el comportamiento real.
- Interpretar cualquier Scan como siempre malo, sin considerar que en tablas muy chicas puede ser mas eficiente que un Seek.
- No transferir el conocimiento ya solido de `EXPLAIN ANALYZE` de Postgres, tratando este tema como completamente nuevo.

## Practica

- [ ] Genera un Execution Plan grafico para una consulta sin indice apropiado.
- [ ] Crea el indice correspondiente y compara el plan resultante.
- [ ] Habilita `SET STATISTICS IO/TIME ON` y compara con la informacion del plan grafico.
- [ ] Identifica la operacion de mayor costo relativo en un plan complejo de multiples tablas.
- [ ] Compara este proceso de diagnostico con el ya dominado con `EXPLAIN ANALYZE` en Postgres.

## Preguntas de repaso

- Que ventaja practica da la representacion grafica de Execution Plans de SQL Server sobre la salida de texto de `EXPLAIN` de Postgres?
- Que operacion equivale a un Seq Scan de Postgres, y cual a un Index Scan eficiente?
- Que diferencia hay entre el plan estimado y el plan real de ejecucion?
- Como usarias el porcentaje de costo relativo para priorizar que optimizar primero en una consulta compleja?
