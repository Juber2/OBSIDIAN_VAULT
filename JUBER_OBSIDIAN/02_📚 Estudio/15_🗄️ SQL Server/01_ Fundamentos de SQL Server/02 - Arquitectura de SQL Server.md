---
tags:
  - SQLServer
  - Bases-de-Datos
  - SQLServer-Fundamentos
Creado: 2026-07-29
Relacionado:
  - Fundamentos de SQL Server
  - SQL SERVER
---

# 02 - Arquitectura de SQL Server

## Que vas a aprender

En este capitulo vas a entender la arquitectura interna de SQL Server: el modelo de **threads** (distinto al modelo de procesos de Postgres, ver libro complementario), la gestion de **memoria** (buffer pool), y el **Transaction Log**, la base de la durabilidad de SQL Server.

## Explicacion

A diferencia de Postgres (que usa un proceso del sistema operativo por cada conexion, ver capitulo de Arquitectura de ese libro), SQL Server usa un modelo de **threads** dentro de un unico proceso del motor (`sqlservr.exe`), con su propio scheduler interno (SQLOS) que gestiona hilos de trabajo de forma mas eficiente que dejarlo completamente al sistema operativo. Esto da a SQL Server un overhead menor por conexion comparado con el modelo de procesos de Postgres, aunque ambos enfoques tienen sus propios tradeoffs de aislamiento y eficiencia.

La memoria se organiza principalmente en el **buffer pool** (equivalente conceptual a `shared_buffers` de Postgres), cacheando paginas de datos en memoria. El **Transaction Log** (`.ldf`) es el equivalente exacto del WAL de Postgres (ver capitulo de ACID de ese libro): cada cambio se escribe primero al log antes de aplicarse a los archivos de datos, garantizando durabilidad ante una caida.

## Modelo mental

Pensa el modelo de threads de SQL Server como un unico restaurante grande con muchos meseros (threads) coordinados por un unico gerente de turno (SQLOS), en vez de que cada cliente (conexion) tenga su propio restaurante independiente completo (el modelo de procesos de Postgres). Ambos enfoques atienden a los clientes, pero con una filosofia distinta de organizacion interna.

## Ejemplo base

```sql
-- Ver informacion de memoria del buffer pool
SELECT * FROM sys.dm_os_memory_clerks WHERE type = 'MEMORYCLERK_SQLBUFFERPOOL';

-- Ver conexiones activas (equivalente a pg_stat_activity de Postgres)
SELECT session_id, login_name, status, program_name
FROM sys.dm_exec_sessions WHERE is_user_process = 1;

-- Configuracion de memoria maxima del servidor
EXEC sp_configure 'max server memory';
```

## Como se usa en un proyecto real

1. Se configura `max server memory` explicitamente en servidores dedicados, evitando que SQL Server consuma toda la memoria disponible del sistema operativo sin dejar margen para otros procesos.
2. Se usa `sys.dm_exec_sessions` (el DMV, Dynamic Management View, equivalente a `pg_stat_activity`) para diagnosticar conexiones activas y su estado.
3. Se entiende que el modelo de threads hace que SQL Server tenga overhead menor por conexion que Postgres, aunque igual se beneficia de connection pooling (ya visto en Entity Framework Core, ver capitulo correspondiente) bajo carga alta.
4. Se conecta el Transaction Log con las mismas garantias de durabilidad ya estudiadas conceptualmente en el libro de PostgreSQL (WAL), entendiendo que es el mismo principio con nombre distinto.
5. Se documenta la configuracion de memoria de servidores de produccion como parte de la infraestructura del proyecto.

## Criterio junior

Un junior debe entender que SQL Server usa un modelo de threads dentro de un unico proceso, distinto del modelo de procesos de Postgres.

## Criterio mid

Un mid debe configurar `max server memory` apropiadamente y usar DMVs basicos para diagnosticar conexiones activas.

## Criterio senior

Un senior debe dimensionar la memoria de servidores SQL Server de produccion considerando otros procesos del sistema, y comparar el modelo de concurrencia con Postgres para decisiones de arquitectura informadas.

## Errores comunes

- No configurar `max server memory`, dejando que SQL Server consuma memoria excesiva y afecte a otros procesos del sistema operativo.
- Asumir que el modelo de threads de SQL Server es identico al de procesos de Postgres, perdiendo matices importantes de cada arquitectura.
- No usar DMVs para diagnostico, perdiendo visibilidad del estado real de conexiones y memoria.
- No documentar la configuracion de memoria de servidores de produccion.

## Practica

- [ ] Configura `max server memory` en una instancia de SQL Server de practica.
- [ ] Consulta `sys.dm_exec_sessions` para ver conexiones activas.
- [ ] Compara conceptualmente el modelo de threads de SQL Server con el modelo de procesos de Postgres.
- [ ] Investiga que es el Transaction Log y como se relaciona con el WAL de Postgres.
- [ ] Documenta la configuracion de memoria que aplicarias a un servidor de produccion.

## Preguntas de repaso

- Que diferencia hay entre el modelo de threads de SQL Server y el modelo de procesos de Postgres?
- Que rol cumple el buffer pool en SQL Server?
- Como se relaciona el Transaction Log de SQL Server con el WAL de Postgres?
- Por que es importante configurar `max server memory` explicitamente?
