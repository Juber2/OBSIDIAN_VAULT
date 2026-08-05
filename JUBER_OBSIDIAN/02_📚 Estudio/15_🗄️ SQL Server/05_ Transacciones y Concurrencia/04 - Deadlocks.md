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

# 04 - Deadlocks

## Que vas a aprender

En este capitulo vas a aplicar la prevencion de deadlocks (ya estudiada en profundidad en el libro de Postgres) especificamente en T-SQL: como SQL Server elige la **victima** del deadlock, y herramientas especificas de diagnostico (Deadlock Graph).

## Explicacion

Un deadlock en SQL Server es conceptualmente identico al ya estudiado en Postgres: dos transacciones bloqueadas mutuamente esperando locks que la otra tiene. SQL Server lo detecta automaticamente y elige una **victima** (la transaccion que aborta) basandose en el **costo estimado de deshacer** cada transaccion (la mas "barata" de revertir generalmente pierde), a diferencia de Postgres que usa un criterio distinto. Se puede influir en esta eleccion con `SET DEADLOCK_PRIORITY` (marcando una transaccion como preferentemente victima o preferentemente sobreviviente).

SQL Server ofrece el **Deadlock Graph**, una representacion visual (disponible en SSMS y capturable via Extended Events) que muestra exactamente que recursos y transacciones estuvieron involucradas en un deadlock especifico, facilitando el diagnostico mucho mas que analizar solo logs de texto.

## Modelo mental

El modelo mental de deadlock es identico al ya establecido en Postgres: dos personas bloqueandose mutuamente el paso en un pasillo. La diferencia especifica de SQL Server es que, ademas de resolver automaticamente el problema (como Postgres), te da un mapa visual detallado (Deadlock Graph) de exactamente que paso, en vez de solo un mensaje de error de texto.

## Ejemplo base

```sql
-- Marcar una transaccion como preferentemente victima si ocurre un deadlock
SET DEADLOCK_PRIORITY LOW;

BEGIN TRANSACTION;
-- operaciones que podrian generar contencion...
COMMIT TRANSACTION;
```

```sql
-- Habilitar captura de Deadlock Graph via Extended Events (configuracion conceptual)
CREATE EVENT SESSION CapturarDeadlocks ON SERVER
ADD EVENT sqlserver.xml_deadlock_report
ADD TARGET package0.event_file(SET filename = 'deadlocks.xel');
```

## Como se usa en un proyecto real

1. Se aplica el mismo orden consistente de acceso a recursos ya establecido en Postgres (por ejemplo, siempre por ID ascendente) para prevenir deadlocks por diseño.
2. Se implementa logica de reintento en la aplicacion .NET (con Polly u otra libreria de resiliencia) para transacciones que fallan con el error especifico de deadlock (numero de error 1205).
3. Se usa `SET DEADLOCK_PRIORITY` en transacciones de bajo impacto (como tareas de mantenimiento en background) que preferentemente deben ser la victima si ocurre contencion con operaciones criticas de usuario.
4. Se captura y analiza el Deadlock Graph ante deadlocks recurrentes en produccion, usando la representacion visual para identificar el patron de acceso problematico.
5. Se conecta esto directamente con el capitulo de Deadlocks del libro de Postgres, transfiriendo el mismo criterio de prevencion por diseño con las herramientas especificas de SQL Server.

## Criterio junior

Un junior debe entender que SQL Server detecta y resuelve deadlocks automaticamente, eligiendo una victima segun costo de reversion.

## Criterio mid

Un mid debe implementar logica de reintento ante el error especifico de deadlock, y aplicar orden consistente de acceso a recursos.

## Criterio senior

Un senior debe analizar Deadlock Graphs para diagnosticar patrones recurrentes de contencion, y usar `DEADLOCK_PRIORITY` estrategicamente para transacciones de distinta criticidad.

## Errores comunes

- No implementar reintentos ante el error de deadlock, dejando que se propague como un fallo no manejado al usuario.
- No aplicar orden consistente de acceso a recursos, generando deadlocks intermitentes dificiles de reproducir.
- No aprovechar el Deadlock Graph para diagnostico, analizando solo mensajes de error de texto menos informativos.
- No transferir el criterio ya solido de prevencion de deadlocks de Postgres a este contexto.

## Practica

- [ ] Provoca un deadlock intencional entre dos transacciones T-SQL y observa como SQL Server elige la victima.
- [ ] Implementa logica de reintento en un servicio .NET ante el error 1205 (deadlock).
- [ ] Aplica orden consistente de acceso a recursos para prevenir un deadlock especifico.
- [ ] Captura (o investiga como capturarias) un Deadlock Graph para un caso simulado.
- [ ] Compara este proceso completo con la prevencion de deadlocks ya dominada en Postgres.

## Preguntas de repaso

- Que criterio usa SQL Server para elegir la victima de un deadlock?
- Que ventaja da el Deadlock Graph sobre analizar solo mensajes de error de texto?
- Como implementarias logica de reintento en .NET ante un deadlock detectado?
- Que practica de diseño (ya vista en Postgres) previene deadlocks tambien en SQL Server?
