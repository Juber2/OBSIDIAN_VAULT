---
tags:
  - SQLServer
  - Transacciones
  - SQLServer-Transacciones
Creado: 2026-07-29
Relacionado:
  - Transacciones y Concurrencia
  - SQL SERVER
---

# 03 - Locks y Lock Escalation

## Que vas a aprender

En este capitulo vas a profundizar en el modelo de **locking** de SQL Server (mucho mas central que en Postgres por default, ver tema anterior), y **Lock Escalation**, un fenomeno especifico de SQL Server sin equivalente en Postgres donde locks de fila se convierten automaticamente en locks de tabla completa.

## Explicacion

SQL Server otorga locks a distintos niveles de granularidad (fila, pagina, tabla) segun lo que una operacion necesita. **Lock Escalation** ocurre cuando una transaccion adquiere **demasiados locks individuales de fila** (por defecto, un umbral aproximado de 5000 locks): en vez de mantener miles de locks pequeños individuales (con el overhead de memoria que eso implica), SQL Server automaticamente los **escala** a un unico lock de tabla completa, bloqueando temporalmente **toda** la tabla para otras transacciones, incluso si la operacion original solo necesitaba modificar una fraccion pequeña de esas filas.

Esto es un fenomeno que sorprende a quien viene de Postgres (sin este comportamiento de escalacion automatica de esa forma): una operacion `UPDATE` masiva pero legitima puede terminar bloqueando la tabla completa inesperadamente, afectando a toda la aplicacion durante ese tiempo, un riesgo real a considerar en operaciones batch grandes.

## Modelo mental

Pensa Lock Escalation como un guardia de seguridad que, al ver que esta emitiendo demasiados pases individuales de acceso a habitaciones especificas de un edificio (locks de fila), decide que es mas simple cerrar el edificio completo con un unico cartel de "cerrado" (lock de tabla) en vez de seguir emitiendo miles de pases individuales, aunque eso signifique que gente que solo queria entrar a otras habitaciones completamente distintas tambien quede bloqueada temporalmente.

## Ejemplo base

```sql
-- Ver locks activos actualmente
SELECT resource_type, resource_database_id, request_mode, request_status
FROM sys.dm_tran_locks WHERE request_session_id = @@SPID;

-- Deshabilitar lock escalation para una tabla especifica (usar con criterio)
ALTER TABLE Pedidos SET (LOCK_ESCALATION = DISABLE);

-- Procesar en lotes pequeños para evitar escalacion en operaciones masivas
DECLARE @Procesados INT = 1;
WHILE @Procesados > 0
BEGIN
    UPDATE TOP (1000) Pedidos SET Estado = 'Procesado' WHERE Estado = 'Pendiente';
    SET @Procesados = @@ROWCOUNT;
END;
```

## Como se usa en un proyecto real

1. Se procesan operaciones masivas en **lotes pequeños** (`UPDATE TOP (N)` en un loop) para evitar acumular suficientes locks de fila como para disparar escalacion, en vez de una unica operacion masiva sobre millones de filas.
2. Se monitorea `sys.dm_tran_locks` al diagnosticar contencion inesperada, buscando evidencia de lock escalation en el momento del problema.
3. Se evalua `LOCK_ESCALATION = DISABLE` solo en casos especificos bien entendidos, dado que deshabilitarlo completamente puede generar el problema opuesto (demasiada memoria consumida en locks individuales).
4. Se conecta esto con el capitulo de Locks de Postgres, entendiendo que este es un fenomeno **especifico de SQL Server** sin equivalente directo, relevante al migrar habitos entre ambos motores.
5. Se documenta cualquier operacion batch que necesito diseñarse en lotes especificamente para evitar lock escalation.

## Criterio junior

Un junior debe entender conceptualmente que es lock escalation y por que puede sorprender a quien viene de Postgres.

## Criterio mid

Un mid debe procesar operaciones masivas en lotes pequeños para evitar disparar escalacion no intencional.

## Criterio senior

Un senior debe diagnosticar problemas de contencion causados por lock escalation en produccion, y decidir cuando ajustar el comportamiento por tabla especifica.

## Errores comunes

- Ejecutar un `UPDATE` masivo sobre millones de filas en una unica operacion, disparando lock escalation y bloqueando la tabla completa inesperadamente.
- No conocer este fenomeno al migrar desde Postgres, sorprendiendose de bloqueos que no ocurrian en ese motor.
- Deshabilitar `LOCK_ESCALATION` globalmente sin entender el tradeoff de memoria que eso implica en operaciones realmente masivas.
- No monitorear `sys.dm_tran_locks` al diagnosticar contencion inesperada en produccion.

## Practica

- [ ] Provoca lock escalation intencionalmente con una operacion masiva sobre muchas filas.
- [ ] Observa el efecto en `sys.dm_tran_locks` durante esa operacion.
- [ ] Rediseña esa operacion en lotes pequeños para evitar la escalacion.
- [ ] Compara este comportamiento con lo que pasaria en Postgres para la misma operacion.
- [ ] Documenta un caso real donde procesarias una operacion masiva en lotes para evitar este problema.

## Preguntas de repaso

- Que es lock escalation y por que ocurre automaticamente en SQL Server?
- Por que este fenomeno sorprende a quien viene de un background de Postgres?
- Como rediseñarias una operacion masiva para evitar disparar lock escalation?
- Que riesgo tiene deshabilitar `LOCK_ESCALATION` completamente sin criterio?
