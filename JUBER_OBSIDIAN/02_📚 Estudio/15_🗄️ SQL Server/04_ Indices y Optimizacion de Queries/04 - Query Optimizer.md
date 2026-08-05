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

# 04 - Query Optimizer

## Que vas a aprender

En este capitulo vas a entender el **Query Optimizer** de SQL Server, equivalente conceptual al query planner de Postgres (ya estudiado en profundidad en ese libro), y **Parameter Sniffing**, un fenomeno especifico de SQL Server sin equivalente directo en Postgres.

## Explicacion

Igual que el planner de Postgres, el Query Optimizer de SQL Server es **basado en costos**, usando estadisticas sobre las tablas para elegir el plan de ejecucion mas eficiente estimado. **Parameter Sniffing** es un fenomeno especifico: cuando un stored procedure o query parametrizada se ejecuta por primera vez, SQL Server **cachea el plan de ejecucion** generado para esos parametros especificos, y reutiliza ese mismo plan cacheado en ejecuciones futuras con parametros distintos, incluso si esos parametros distintos hubieran generado un plan diferente (y potencialmente mejor) si se hubieran optimizado desde cero.

Esto causa un problema real y confuso: una consulta que "funciona rapido" con ciertos valores de parametro puede volverse lenta con otros valores, no porque el dato en si sea distinto, sino porque el plan cacheado no es optimo para esos parametros especificos. Postgres tiene un comportamiento de cacheo de planes mas conservador y menos propenso a este problema especifico en su forma clasica.

## Modelo mental

Pensa Parameter Sniffing como un mesero que aprende una unica ruta optima para llevar pedidos a la mesa 5 la primera vez que trabaja ahi, y despues sigue usando exactamente esa misma ruta memorizada para cualquier mesa, incluso cuando en realidad estaba llevando el pedido a la mesa 20 (un caso completamente distinto donde esa ruta memorizada ya no es la optima).

## Ejemplo base

```sql
CREATE PROCEDURE ObtenerPedidosPorCliente @ClienteId BIGINT
AS
BEGIN
    SELECT * FROM Pedidos WHERE ClienteId = @ClienteId;
END;

-- Primera ejecucion con un cliente que tiene pocos pedidos: genera y cachea un plan optimo para ESE caso
EXEC ObtenerPedidosPorCliente @ClienteId = 5;

-- Ejecucion posterior con un cliente que tiene millones de pedidos: reusa el plan cacheado (subóptimo)
EXEC ObtenerPedidosPorCliente @ClienteId = 999;

-- Forzar recompilacion del plan, ignorando el cache (usar con criterio, no por defecto)
EXEC ObtenerPedidosPorCliente @ClienteId = 999 WITH RECOMPILE;

-- Ver el plan cacheado actual de un stored procedure
SELECT * FROM sys.dm_exec_cached_plans WHERE objtype = 'Proc';
```

## Como se usa en un proyecto real

1. Se identifica Parameter Sniffing como sospechoso cuando una consulta parametrizada o stored procedure "a veces es rapida y a veces lenta" con los mismos parametros tipicos pero valores distintos.
2. Se usa `WITH RECOMPILE` selectivamente en stored procedures donde la distribucion de valores de parametros es muy variable, aceptando el costo adicional de recompilar en cada ejecucion.
3. Se considera `OPTION (OPTIMIZE FOR UNKNOWN)` como alternativa que genera un plan basado en estadisticas generales en vez de los parametros de la primera ejecucion especifica.
4. Se conecta esto con el concepto ya dominado de estadisticas y `ANALYZE` de Postgres, entendiendo que ambos motores dependen de estadisticas actualizadas, pero SQL Server tiene esta particularidad adicional de cacheo de plan por parametros.
5. Se documenta cualquier stored procedure con `WITH RECOMPILE` explicando la razon, dado que no es la configuracion por defecto esperada.

## Criterio junior

Un junior debe entender conceptualmente que es Parameter Sniffing y por que puede causar performance inconsistente.

## Criterio mid

Un mid debe diagnosticar un caso de Parameter Sniffing revisando el plan cacheado y comparando con el comportamiento esperado.

## Criterio senior

Un senior debe decidir la estrategia apropiada (`RECOMPILE`, `OPTIMIZE FOR UNKNOWN`, o rediseño del stored procedure) para mitigar Parameter Sniffing en casos criticos de produccion.

## Errores comunes

- No reconocer Parameter Sniffing como causa de performance inconsistente, atribuyendo el problema erroneamente a otra cosa.
- Usar `WITH RECOMPILE` por defecto en todos los stored procedures, sumando overhead de recompilacion innecesario en casos donde no hay problema real de sniffing.
- No revisar el plan cacheado al diagnosticar un stored procedure con comportamiento variable.
- Confundir Parameter Sniffing con problemas de estadisticas desactualizadas, que son causas distintas aunque relacionadas.

## Practica

- [ ] Crea un stored procedure parametrizado y observa el plan cacheado despues de la primera ejecucion.
- [ ] Simula el problema ejecutando con parametros muy distintos en volumen de datos esperado.
- [ ] Aplica `WITH RECOMPILE` y compara el comportamiento.
- [ ] Investiga `OPTION (OPTIMIZE FOR UNKNOWN)` como alternativa.
- [ ] Documenta un caso real (o hipotetico) de Parameter Sniffing y la solucion que aplicarias.

## Preguntas de repaso

- Que es Parameter Sniffing y por que puede causar que una consulta sea rapida a veces y lenta otras?
- Que hace `WITH RECOMPILE` y que costo tiene aplicarlo?
- Como diagnosticarias si un problema de performance variable se debe a Parameter Sniffing?
- Que alternativa a `RECOMPILE` existe para mitigar este problema sin recompilar en cada ejecucion?
