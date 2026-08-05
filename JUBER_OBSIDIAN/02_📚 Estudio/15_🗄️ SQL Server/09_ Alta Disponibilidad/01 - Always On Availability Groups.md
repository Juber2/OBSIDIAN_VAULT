---
tags:
  - SQLServer
  - AltaDisponibilidad
  - SQLServer-AltaDisponibilidad
Creado: 2026-07-29
Relacionado:
  - Alta Disponibilidad
  - SQL SERVER
---

# 01 - Always On Availability Groups

## Que vas a aprender

En este capitulo vas a aprender **Always On Availability Groups (AG)**, la solucion de alta disponibilidad principal de SQL Server moderno, conceptualmente equivalente a la replicacion streaming de Postgres con failover automatico ya estudiada en ese libro.

## Explicacion

Un **Availability Group** agrupa una o mas bases de datos que se replican juntas como una unidad hacia una o mas **replicas secundarias**, cada replica alojada en una instancia distinta de SQL Server (tipicamente en nodos distintos, formando parte de un **Windows Server Failover Cluster** o, en versiones modernas, un cluster basado en Linux Pacemaker). La replica **primaria** procesa todas las escrituras; las replicas **secundarias** reciben los cambios via un mecanismo de envio de log similar en espiritu al streaming replication de Postgres, y opcionalmente pueden aceptar lecturas (**readable secondaries**), el equivalente directo a las read replicas de Postgres.

El modo de disponibilidad puede ser **synchronous commit** (la transaccion en el primario espera confirmacion de la secundaria antes de dar por terminada la escritura, garantizando cero perdida de datos ante failover, con costo de latencia) o **asynchronous commit** (el primario no espera, menor latencia pero riesgo de perdida de datos recientes ante un failover), la misma disyuntiva ya analizada entre replicacion sincrona y asincrona en el libro de Postgres.

## Modelo mental

Pensa un Availability Group como un equipo de escribas donde uno (el primario) escribe el documento oficial y los demas (secundarios) copian cada linea a medida que se escribe, listos para tomar el rol de escriba principal si el actual se ausenta de repente. El modo synchronous es como exigir que al menos un copista confirme haber copiado la linea antes de considerarla oficialmente escrita; asynchronous es dejar que el escriba principal siga sin esperar confirmacion, aceptando que si algo pasa justo en ese instante, la ultima linea podria perderse.

## Ejemplo base

```sql
-- Crear un Availability Group (simplificado, requiere cluster subyacente configurado)
CREATE AVAILABILITY GROUP MiAG
    FOR DATABASE MiApp
    REPLICA ON
        'SQLNODE1' WITH (
            ENDPOINT_URL = 'TCP://sqlnode1:5022',
            AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
            FAILOVER_MODE = AUTOMATIC),
        'SQLNODE2' WITH (
            ENDPOINT_URL = 'TCP://sqlnode2:5022',
            AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
            FAILOVER_MODE = AUTOMATIC);

-- Verificar el estado de sincronizacion
SELECT database_name, synchronization_state_desc
FROM sys.dm_hadr_database_replica_states;
```

## Como se usa en un proyecto real

1. Se configura un Availability Group con failover automatico para bases de datos criticas de produccion, garantizando continuidad ante la caida de un nodo.
2. Se usa `synchronous commit` para el nodo de failover automatico (cero perdida de datos) y opcionalmente `asynchronous commit` para replicas adicionales de solo lectura o en otra region geografica.
3. Se dirige trafico de solo lectura (reportes, dashboards) hacia replicas secundarias legibles, descargando al primario, el mismo patron de read replicas ya visto en Postgres.
4. Se conecta esto directamente con streaming replication y failover automatico (Patroni) del libro de Postgres, entendiendo que resuelven el mismo problema con arquitecturas paralelas.
5. Se prueba el proceso de failover regularmente en un ambiente controlado, no solo se confia en que "deberia funcionar" ante un incidente real.

## Criterio junior

Un junior debe entender que es un Availability Group y la diferencia entre replica primaria y secundaria.

## Criterio mid

Un mid debe configurar un AG con modo de disponibilidad apropiado y dirigir lecturas hacia replicas secundarias.

## Criterio senior

Un senior debe diseñar la arquitectura completa de alta disponibilidad (numero de replicas, modo sincrono/asincrono, distribucion geografica) y probar el failover regularmente.

## Errores comunes

- Usar `asynchronous commit` en el nodo de failover automatico, arriesgando perdida de datos que el equipo asumia protegida.
- No dirigir trafico de lectura hacia replicas secundarias, desperdiciando capacidad de hardware disponible.
- No probar el proceso de failover regularmente, descubriendo problemas de configuracion durante un incidente real.
- Subestimar la complejidad de configurar el cluster subyacente (WSFC o Pacemaker) necesario para que un AG funcione.

## Practica

- [ ] Investiga los requisitos de infraestructura para un Availability Group (cluster subyacente).
- [ ] Compara `synchronous commit` vs `asynchronous commit` en terminos de latencia y garantia de datos.
- [ ] Diseña una arquitectura de AG para un sistema de produccion hipotetico con requisitos de alta disponibilidad.
- [ ] Investiga como dirigir trafico de solo lectura hacia replicas secundarias legibles.
- [ ] Compara Always On AG con streaming replication + Patroni de Postgres.

## Preguntas de repaso

- Que diferencia hay entre `synchronous commit` y `asynchronous commit` en un Availability Group?
- Que es una readable secondary y para que sirve?
- Como se relaciona Always On AG con streaming replication de Postgres?
- Que infraestructura subyacente necesita un Availability Group para funcionar?
