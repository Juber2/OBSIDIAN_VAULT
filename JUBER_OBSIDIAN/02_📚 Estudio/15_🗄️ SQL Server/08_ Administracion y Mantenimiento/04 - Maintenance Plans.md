---
tags:
  - SQLServer
  - Administracion
  - SQLServer-Administracion
Creado: 2026-07-29
Relacionado:
  - Administracion y Mantenimiento
  - SQL SERVER
---

# 04 - Maintenance Plans

## Que vas a aprender

En este capitulo vas a aprender **Maintenance Plans** de SQL Server: reconstruccion/reorganizacion de indices fragmentados (equivalente conceptual a `VACUUM` de Postgres) y actualizacion de estadisticas, empaquetados como tareas automatizables.

## Explicacion

Con el tiempo, las escrituras (`INSERT`/`UPDATE`/`DELETE`) fragmentan los indices de SQL Server, degradando su eficiencia de lectura, un problema conceptualmente relacionado con el table bloat de Postgres (ver capitulo de MVCC de ese libro), aunque el mecanismo subyacente es distinto (fragmentacion de paginas de indice, no tuplas muertas por MVCC). `ALTER INDEX ... REORGANIZE` desfragmenta de forma liviana (sin bloquear significativamente), mientras `ALTER INDEX ... REBUILD` reconstruye el indice completo (mas efectivo, pero puede requerir mas recursos y bloqueo, mitigable con la opcion `ONLINE` en ediciones que lo soportan).

Un **Maintenance Plan** (configurable visualmente en SSMS, o como un job de SQL Server Agent, ver tema siguiente) automatiza: reconstruccion/reorganizacion de indices segun su nivel de fragmentacion, actualizacion de estadisticas (ya visto en el capitulo de Indices), y verificacion de integridad (`DBCC CHECKDB`), ejecutandose en una ventana de mantenimiento programada regularmente.

## Modelo mental

Pensa la fragmentacion de indices como los cajones de un archivo fisico que, despues de mucho uso (agregar y quitar documentos constantemente), quedan con espacios desordenados y huecos dispersos entre las carpetas. `REORGANIZE` es como ordenar rapidamente sin sacar todo del cajon, mientras `REBUILD` es vaciar el cajon completo y volver a acomodarlo desde cero, mas prolijo pero mas disruptivo mientras se hace.

## Ejemplo base

```sql
-- Ver el nivel de fragmentacion de indices de una tabla
SELECT i.name, ips.avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('Pedidos'), NULL, NULL, 'LIMITED') ips
JOIN sys.indexes i ON i.object_id = ips.object_id AND i.index_id = ips.index_id;

-- Reorganizar (liviano, fragmentacion moderada, 5-30%)
ALTER INDEX IX_Pedidos_ClienteId ON Pedidos REORGANIZE;

-- Reconstruir (mas efectivo, fragmentacion alta, >30%)
ALTER INDEX IX_Pedidos_ClienteId ON Pedidos REBUILD;

-- Verificar integridad de la base de datos
DBCC CHECKDB('MiApp');
```

## Como se usa en un proyecto real

1. Se monitorea el nivel de fragmentacion regularmente, aplicando `REORGANIZE` para fragmentacion moderada (5-30%) y `REBUILD` para fragmentacion alta (>30%), la regla practica estandar de la industria.
2. Se configura un Maintenance Plan (o Agent Job equivalente, ver tema siguiente) que automatiza esta tarea en una ventana de mantenimiento de bajo trafico, aplicando el mismo principio ya visto con `autovacuum` de Postgres.
3. Se ejecuta `DBCC CHECKDB` periodicamente para verificar la integridad fisica de la base de datos, detectando corrupcion antes de que cause problemas mayores.
4. Se conecta esto con el capitulo de VACUUM del libro de Postgres, entendiendo que ambos motores necesitan mantenimiento regular de estructuras internas, aunque con mecanismos distintos.
5. Se documenta el proceso de mantenimiento automatizado como parte de la infraestructura del proyecto.

## Criterio junior

Un junior debe poder verificar el nivel de fragmentacion de un indice y ejecutar `REORGANIZE`/`REBUILD` manualmente.

## Criterio mid

Un mid debe automatizar el mantenimiento de indices y estadisticas segun umbrales de fragmentacion apropiados.

## Criterio senior

Un senior debe diseñar la ventana de mantenimiento completa de sistemas de produccion criticos, balanceando frecuencia de mantenimiento con impacto en disponibilidad.

## Errores comunes

- No monitorear la fragmentacion de indices regularmente, dejando que degrade performance de lectura silenciosamente con el tiempo.
- Usar `REBUILD` indiscriminadamente cuando `REORGANIZE` seria suficiente y menos disruptivo para fragmentacion moderada.
- No automatizar el mantenimiento, dependiendo de ejecucion manual que puede olvidarse.
- No ejecutar `DBCC CHECKDB` periodicamente, perdiendo la oportunidad de detectar corrupcion temprano.

## Practica

- [ ] Consulta el nivel de fragmentacion de indices de una tabla real o de practica.
- [ ] Ejecuta `REORGANIZE` para fragmentacion moderada y `REBUILD` para fragmentacion alta simulada.
- [ ] Ejecuta `DBCC CHECKDB` en una base de datos de practica.
- [ ] Diseña un Maintenance Plan (o Agent Job) que automatice este proceso.
- [ ] Compara este mantenimiento con `VACUUM`/`autovacuum` ya estudiado en Postgres.

## Preguntas de repaso

- Que diferencia hay entre `REORGANIZE` y `REBUILD` de un indice?
- Que regla practica usarias para decidir entre uno u otro segun el nivel de fragmentacion?
- Para que sirve `DBCC CHECKDB`?
- Como se relaciona este mantenimiento con `VACUUM`/`autovacuum` de Postgres, y en que se diferencia el mecanismo subyacente?
