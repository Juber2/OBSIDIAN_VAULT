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

# 03 - Failover con Patroni

## Que vas a aprender

En este capitulo vas a aprender **Patroni**, la herramienta estandar de la industria para **failover automatico** de Postgres, conectando directamente con los conceptos de alta disponibilidad ya vistos en el libro de Linux (Keepalived, IP virtual).

## Explicacion

Streaming Replication (ya visto) mantiene replicas sincronizadas, pero **no incluye** un mecanismo de failover automatico: si el primario falla, alguien (o algo) tiene que promover manualmente una replica a nuevo primario y redirigir el trafico. **Patroni** automatiza completamente ese proceso: monitorea continuamente la salud del primario, y ante una falla detectada, **elige automaticamente** la replica mas actualizada, la **promueve** a nuevo primario, y reconfigura las demas replicas para seguir al nuevo primario, todo sin intervencion manual.

Patroni usa un almacen de configuracion distribuido (**etcd**, **Consul**, o **ZooKeeper**) como fuente de verdad compartida sobre "quien es el primario actual", evitando el problema de **split-brain** (dos nodos creyendo ser primario simultaneamente, un riesgo serio de corrupcion de datos). Se combina tipicamente con un proxy de conexion (como **HAProxy**, ya visto en el libro de Linux) que siempre enruta trafico de escritura hacia el primario actual segun lo que Patroni reporta.

## Modelo mental

Pensa Patroni como un sistema de sucesion automatica de una monarquia: en vez de dejar que la corte decida caoticamente quien sucede al rey cuando este cae enfermo (failover manual), hay un protocolo automatico y acordado de antemano que consulta un registro oficial compartido (etcd) para determinar sin ambiguedad quien es el heredero legitimo, evitando que dos personas reclamen el trono simultaneamente (split-brain).

## Ejemplo base

```yaml
# patroni.yml (configuracion conceptual simplificada)
scope: mi-cluster-postgres
restapi:
  listen: 0.0.0.0:8008
etcd:
  hosts: etcd1:2379,etcd2:2379,etcd3:2379
postgresql:
  data_dir: /var/lib/postgresql/data
  parameters:
    max_connections: 200
```

```bash
patronictl -c patroni.yml list        # muestra el estado del cluster: quien es primario, replicas
patronictl -c patroni.yml switchover   # failover manual controlado (mantenimiento planificado)
patronictl -c patroni.yml failover     # failover forzado
```

## Como se usa en un proyecto real

1. Se evalua Patroni cuando la disponibilidad de la base de datos es genuinamente critica y justifica la complejidad operativa adicional, aplicando el mismo marco de decision de arquitectura ya visto en el libro de Linux.
2. Se combina Patroni con HAProxy (o un proxy equivalente) que siempre redirige trafico de escritura hacia el primario actual reportado por Patroni.
3. Se usa `switchover` (controlado) en vez de `failover` (de emergencia) para mantenimientos planificados, minimizando riesgo.
4. Se monitorea activamente el cluster de etcd/Consul usado por Patroni, dado que se convierte en una dependencia critica del sistema completo.
5. Se conecta esto directamente con el capitulo de Alta Disponibilidad y Clustering del libro de Linux, donde el concepto general de failover ya se estudio con Keepalived a nivel de infraestructura mas general.

## Criterio junior

Un junior debe entender conceptualmente que hace Patroni (failover automatico) y por que Streaming Replication por si sola no lo incluye.

## Criterio mid

Un mid debe usar `patronictl` para inspeccionar el estado de un cluster y ejecutar un `switchover` controlado.

## Criterio senior

Un senior debe decidir cuando la complejidad de Patroni se justifica para un sistema especifico, diseñar la arquitectura completa (Patroni + etcd + HAProxy), y liderar simulacros de failover para validar el proceso.

## Errores comunes

- Adoptar Patroni sin necesidad real, agregando complejidad operativa significativa (etcd, HAProxy, coordinacion) sin un requisito de disponibilidad que lo justifique.
- No monitorear el almacen distribuido (etcd/Consul) del que Patroni depende, convirtiendolo en un punto de fallo no vigilado.
- Usar `failover` forzado en vez de `switchover` controlado para mantenimientos planificados, arriesgando una transicion mas abrupta de lo necesario.
- No simular failovers regularmente, descubriendo problemas de configuracion recien durante un incidente real.

## Practica

- [ ] Investiga la arquitectura completa de un cluster Patroni (Postgres + etcd + HAProxy).
- [ ] Si es posible, configura un cluster Patroni basico de practica con dos o tres nodos.
- [ ] Ejecuta `patronictl list` para ver el estado del cluster.
- [ ] Practica un `switchover` controlado y observa la transicion.
- [ ] Evalua si un sistema real que conozcas justificaria la complejidad de Patroni.

## Preguntas de repaso

- Que problema resuelve Patroni que Streaming Replication por si sola no resuelve?
- Que es split-brain y como lo previene Patroni usando un almacen distribuido como etcd?
- Que diferencia hay entre `switchover` y `failover` en Patroni?
- Que factores considerarias antes de adoptar Patroni para un sistema especifico?
