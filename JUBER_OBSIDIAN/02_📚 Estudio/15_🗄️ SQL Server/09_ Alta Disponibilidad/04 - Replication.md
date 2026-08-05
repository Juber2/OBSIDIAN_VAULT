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

# 04 - Replication

## Que vas a aprender

En este capitulo vas a aprender **SQL Server Replication** (Transactional, Merge, Snapshot), un mecanismo distinto en proposito a Always On AG/Log Shipping: en vez de mantener replicas identicas para alta disponibilidad, distribuye subconjuntos especificos de datos entre servidores, equivalente conceptual a la replicacion logica de Postgres ya estudiada en ese libro.

## Explicacion

**Transactional Replication** distribuye cambios casi en tiempo real desde un **Publisher** hacia uno o mas **Subscribers**, tipicamente para escenarios donde solo ciertas tablas o columnas necesitan distribuirse (no la base de datos completa), como alimentar un data warehouse de reportes o sincronizar un subconjunto de datos hacia una sucursal remota, el mismo caso de uso que la replicacion logica basada en publicacion/suscripcion de Postgres. **Merge Replication** permite cambios en **ambas direcciones** (Publisher y Subscriber pueden escribir), con resolucion de conflictos configurable, util para escenarios de sincronizacion offline/mobile donde un Subscriber puede desconectarse y reconectarse despues. **Snapshot Replication** simplemente copia el estado completo periodicamente, apropiado para datos que cambian poco y donde la latencia de sincronizacion no importa.

La diferencia clave frente a Always On AG es el proposito: AG replica **toda la base de datos identica** para continuidad de servicio; Replication distribuye **subconjuntos de datos** con transformaciones posibles, para integracion entre sistemas o distribucion selectiva, no para alta disponibilidad.

## Modelo mental

Pensa Always On AG como fotocopiar el libro completo para tener un respaldo identico listo para reemplazar el original. Pensa Replication como recortar y enviar solo los capitulos relevantes de ese libro a distintos departamentos que solo necesitan esa parte, algunos de los cuales (Merge Replication) incluso pueden escribir anotaciones que vuelven al libro original.

## Ejemplo base

```sql
-- Replication se configura principalmente via SSMS o stored procedures de sistema.
-- Ejemplo simplificado de configuracion de una publicacion transaccional:

EXEC sp_replicationdboption
    @dbname = 'MiApp',
    @optname = 'publish',
    @value = 'true';

EXEC sp_addpublication
    @publication = 'PublicacionPedidos',
    @status = 'active';

EXEC sp_addarticle
    @publication = 'PublicacionPedidos',
    @article = 'Pedidos',
    @source_object = 'Pedidos';
```

## Como se usa en un proyecto real

1. Se usa Transactional Replication para alimentar un data warehouse de reportes con datos casi en tiempo real, sin sobrecargar la base de datos transaccional principal con queries analiticas pesadas.
2. Se usa Merge Replication en escenarios donde sucursales remotas o dispositivos moviles necesitan trabajar offline y sincronizar cambios bidireccionales al reconectarse.
3. Se distingue claramente Replication (distribucion de subconjuntos de datos) de Always On AG (alta disponibilidad de la base de datos completa), evitando confundir ambas herramientas para el proposito equivocado.
4. Se conecta esto con la replicacion logica de Postgres (publication/subscription), entendiendo que resuelven el mismo problema de distribucion selectiva de datos.
5. Se documenta el esquema de replicacion (que tablas, en que direccion, con que resolucion de conflictos) como parte de la arquitectura de integracion de datos.

## Criterio junior

Un junior debe entender que Replication distribuye subconjuntos de datos, a diferencia de AG que replica la base de datos completa para alta disponibilidad.

## Criterio mid

Un mid debe configurar Transactional Replication para un escenario de distribucion de datos hacia un sistema de reportes.

## Criterio senior

Un senior debe diseñar arquitecturas de integracion de datos usando el tipo de Replication apropiado (Transactional, Merge, Snapshot) segun los requisitos de direccionalidad y latencia.

## Errores comunes

- Confundir Replication con una solucion de alta disponibilidad, cuando en realidad su proposito es distribucion selectiva de datos.
- Usar Merge Replication sin definir una estrategia clara de resolucion de conflictos, generando inconsistencias dificiles de depurar.
- No considerar la carga adicional que Replication genera sobre el servidor Publisher, especialmente con muchos Subscribers.
- Elegir Replication cuando el problema real era alta disponibilidad, que Always On AG resuelve mejor.

## Practica

- [ ] Investiga la diferencia entre Transactional, Merge, y Snapshot Replication.
- [ ] Diseña un escenario de Transactional Replication para alimentar un data warehouse de reportes.
- [ ] Investiga como se configura la resolucion de conflictos en Merge Replication.
- [ ] Compara el proposito de Replication con el de Always On AG, evitando confundirlos.
- [ ] Compara esta tecnica con la replicacion logica (publication/subscription) de Postgres.

## Preguntas de repaso

- Que diferencia de proposito hay entre Replication y Always On AG?
- En que escenario usarias Merge Replication en vez de Transactional Replication?
- Que riesgo introduce Merge Replication que Transactional Replication no tiene?
- Como se relaciona SQL Server Replication con la replicacion logica de Postgres?
