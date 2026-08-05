---
tags:
  - PostgreSQL
  - Alta-Disponibilidad
  - Postgres-HA
Creado: 2026-07-29
Relacionado:
  - Replicacion y Alta Disponibilidad
  - POSTGRESQL
---

# 01 - Streaming Replication

## Que vas a aprender

En este capitulo vas a aprender **Streaming Replication**, el mecanismo nativo de Postgres para mantener una o mas **replicas** sincronizadas en tiempo casi real con un servidor **primario**, la base tecnica de alta disponibilidad y read replicas.

## Explicacion

Streaming Replication aprovecha el mismo WAL ya visto en el capitulo de PITR: en vez de solo archivar el WAL para recuperacion futura, el primario **transmite** continuamente sus cambios a uno o mas servidores **replica**, que los aplican en tiempo casi real, manteniendo una copia constantemente actualizada de la base de datos. Puede ser **sincrona** (el primario espera confirmacion de que la replica recibio el cambio antes de confirmar la transaccion al cliente, garantizando cero perdida de datos ante un fallo del primario, a costa de latencia adicional) o **asincrona** (el primario confirma inmediatamente sin esperar a la replica, mas rapido pero con una ventana pequeña de posible perdida de datos si el primario falla justo despues de confirmar).

Una replica en Streaming Replication puede configurarse en **hot standby**: acepta conexiones de solo lectura mientras sigue replicando, permitiendo distribuir carga de lectura entre el primario y sus replicas (ver siguiente capitulo, Read Replicas).

## Modelo mental

Pensa Streaming Replication como una transmision en vivo continua: el primario "narra" cada cambio que hace en tiempo real, y la replica va reproduciendo esa narracion para mantenerse al dia con lo que esta pasando. Replicacion sincrona es como esperar la confirmacion de que el oyente realmente escucho cada frase antes de continuar con la siguiente; asincrona es seguir narrando sin esperar esa confirmacion, confiando en que la mayoria del tiempo llega bien.

## Ejemplo base

```
# postgresql.conf del primario
wal_level = replica
max_wal_senders = 5

# pg_hba.conf del primario: permitir conexion de replicacion desde la replica
host replication replicador 10.0.0.20/32 scram-sha-256
```

```bash
# En el servidor replica: clonar el primario e iniciar en modo standby
pg_basebackup -h primario.interno -U replicador -D /var/lib/postgresql/data -R
# el flag -R crea automaticamente la configuracion de standby necesaria
```

```sql
-- Verificar el estado de replicacion desde el primario
SELECT client_addr, state, sync_state FROM pg_stat_replication;
```

## Como se usa en un proyecto real

1. Se configura al menos una replica en Streaming Replication asincrona como base minima de alta disponibilidad para cualquier sistema critico en produccion.
2. Se evalua replicacion sincrona solo para datos donde la perdida cero es absolutamente critica (transacciones financieras), aceptando la latencia adicional que introduce.
3. Se usa `pg_stat_replication` en el primario para monitorear el estado y retraso de cada replica conectada.
4. Se conecta esto con el capitulo de Alta Disponibilidad del libro de Linux, donde este mismo concepto de failover se aplico de forma mas general a infraestructura completa.
5. Se documenta la topologia de replicacion (cuantas replicas, sincronas o asincronas, en que region) como parte de la arquitectura de infraestructura del sistema.

## Criterio junior

Un junior debe entender la diferencia conceptual entre replicacion sincrona y asincrona, y poder configurar una replica basica con `pg_basebackup -R`.

## Criterio mid

Un mid debe monitorear el estado de replicas con `pg_stat_replication`, y decidir cuando la replicacion sincrona se justifica frente al costo de latencia.

## Criterio senior

Un senior debe diseñar la topologia de replicacion completa de un sistema critico, balanceando sincrona/asincrona segun el dato especifico, y coordinar con la estrategia de alta disponibilidad general de la infraestructura.

## Errores comunes

- No tener ninguna replica configurada en un sistema critico de produccion, dejando el primario como unico punto de fallo sin capacidad de failover rapido.
- Usar replicacion sincrona para todo el sistema sin necesidad real, introduciendo latencia innecesaria en operaciones que no la requieren.
- No monitorear el retraso (lag) de replicas asincronas, arriesgando una ventana de perdida de datos mayor a la aceptable ante un fallo del primario.
- No documentar la topologia de replicacion, dificultando el entendimiento del sistema para nuevos integrantes del equipo.

## Practica

- [ ] Configura una replica basica en Streaming Replication asincrona con `pg_basebackup -R`.
- [ ] Verifica el estado de la replicacion desde el primario con `pg_stat_replication`.
- [ ] Simula una escritura en el primario y verifica que aparece en la replica.
- [ ] Investiga como configurarias replicacion sincrona y el tradeoff de latencia que introduce.
- [ ] Documenta la topologia de replicacion que diseñarias para un sistema critico hipotetico.

## Preguntas de repaso

- Que diferencia hay entre replicacion sincrona y asincrona en terminos de garantia de datos y latencia?
- Que rol cumple el WAL en Streaming Replication, conectando con lo ya visto en PITR?
- Como verificarias el estado y retraso de una replica desde el primario?
- Cuando se justificaria usar replicacion sincrona pese a su costo de latencia?
