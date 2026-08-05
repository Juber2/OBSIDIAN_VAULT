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

# 02 - Query Store

## Que vas a aprender

En este capitulo vas a aprender **Query Store**, una funcionalidad sin equivalente directo en Postgres estandar (que se aproxima con la extension externa `pg_stat_statements` ya vista en ese libro): SQL Server captura y persiste automaticamente el historial de queries ejecutadas, sus planes de ejecucion, y sus metricas de performance a lo largo del tiempo.

## Explicacion

Una vez habilitado (`ALTER DATABASE ... SET QUERY_STORE = ON`), Query Store registra cada query ejecutada junto con su plan de ejecucion y estadisticas de recursos consumidos (duracion, CPU, I/O), permitiendo comparar el comportamiento de la **misma query a lo largo del tiempo**, algo que un Execution Plan aislado (capturado en un momento puntual) no puede mostrar. Esto es particularmente valioso para detectar **plan regressions**: cuando SQL Server elige un plan de ejecucion nuevo (por ejemplo, tras una actualizacion de estadisticas o un cambio de parametro, ver Parameter Sniffing) que resulta ser mas lento que el plan anterior usado historicamente para la misma query.

Query Store permite **forzar un plan especifico** (`sp_query_store_force_plan`) cuando se identifica que un plan historico era mejor que el actual, una capacidad de intervencion directa que va mas alla de lo que ofrece `pg_stat_statements` (que es puramente informativo, sin capacidad de forzar planes).

## Modelo mental

Pensa Query Store como una caja negra de avion para la base de datos: graba automaticamente cada "vuelo" (ejecucion de query) junto con la "ruta" que tomo (el plan de ejecucion) y cuanto "combustible" consumio (recursos). Si un vuelo reciente fue mas lento que vuelos anteriores de la misma ruta, podes revisar la caja negra para entender exactamente que cambio, y hasta forzar que futuros vuelos usen la ruta que historicamente funciono mejor.

## Ejemplo base

```sql
-- Habilitar Query Store
ALTER DATABASE MiApp SET QUERY_STORE = ON;

-- Ver las queries con mayor consumo de recursos historicamente
SELECT q.query_id, qt.query_sql_text, rs.avg_duration
FROM sys.query_store_query q
JOIN sys.query_store_query_text qt ON q.query_text_id = qt.query_text_id
JOIN sys.query_store_plan p ON q.query_id = p.query_id
JOIN sys.query_store_runtime_stats rs ON p.plan_id = rs.plan_id
ORDER BY rs.avg_duration DESC;

-- Forzar un plan especifico que historicamente funciono mejor
EXEC sp_query_store_force_plan @query_id = 42, @plan_id = 17;
```

## Como se usa en un proyecto real

1. Se habilita Query Store en toda base de datos de produccion de SQL Server 2016+ (StaffTrainingPlatform incluido), dado su bajo costo y alto valor diagnostico.
2. Se revisa regularmente el reporte de "Regressed Queries" (disponible visualmente en SSMS) para detectar queries que empeoraron su performance recientemente.
3. Se usa `sp_query_store_force_plan` como solucion temporal ante una plan regression detectada en produccion, mientras se investiga la causa raiz (frecuentemente Parameter Sniffing).
4. Se conecta esto con `pg_stat_statements` de Postgres, entendiendo que resuelven un problema similar (visibilidad historica de queries) pero Query Store añade capacidad de intervencion directa sobre planes.
5. Se usa el historico de Query Store como evidencia objetiva al reportar problemas de performance a stakeholders, mostrando la tendencia real en vez de una medicion puntual.

## Criterio junior

Un junior debe poder habilitar Query Store y consultar las queries con mayor consumo de recursos.

## Criterio mid

Un mid debe identificar plan regressions usando el historico de Query Store y entender su causa probable.

## Criterio senior

Un senior debe usar `sp_query_store_force_plan` con criterio como mitigacion temporal, mientras diseña una solucion definitiva a la causa raiz de una regresion de performance.

## Errores comunes

- No habilitar Query Store en produccion, perdiendo visibilidad historica valiosa sobre el comportamiento real de las queries.
- Forzar un plan con `sp_query_store_force_plan` como solucion permanente sin investigar ni resolver la causa raiz de la regresion.
- No revisar el reporte de Regressed Queries regularmente, dejando pasar degradaciones de performance silenciosas.
- Confundir Query Store con `pg_stat_statements`, sin reconocer la capacidad adicional de forzar planes que Query Store ofrece.

## Practica

- [ ] Habilita Query Store en una base de datos de practica.
- [ ] Ejecuta una query varias veces y consulta su historico de performance en Query Store.
- [ ] Provoca intencionalmente una plan regression (por ejemplo, forzando un plan subptimo) y detectala usando Query Store.
- [ ] Usa `sp_query_store_force_plan` para forzar un plan mejor.
- [ ] Compara Query Store con `pg_stat_statements` de Postgres, identificando la capacidad adicional que ofrece.

## Preguntas de repaso

- Que problema resuelve Query Store que un Execution Plan aislado no puede mostrar?
- Que es una plan regression, y como la detectarias con Query Store?
- Que capacidad tiene Query Store que `pg_stat_statements` de Postgres no ofrece?
- Por que forzar un plan deberia considerarse una mitigacion temporal y no una solucion definitiva?
