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

# 02 - Failover Clustering

## Que vas a aprender

En este capitulo vas a aprender **Failover Cluster Instances (FCI)**, una solucion de alta disponibilidad a nivel de **instancia completa** (no a nivel de base de datos individual como Always On AG), basada en almacenamiento compartido.

## Explicacion

A diferencia de Always On AG (donde cada replica tiene su propia copia fisica de los datos, sincronizada por envio de log), un **Failover Cluster Instance** usa **almacenamiento compartido** (tipicamente un SAN): solo un nodo del cluster tiene la instancia de SQL Server activa en un momento dado, accediendo al mismo almacenamiento fisico; ante la falla del nodo activo, el cluster (Windows Server Failover Cluster) mueve automaticamente la instancia completa a otro nodo, que monta el mismo almacenamiento compartido y retoma el servicio.

La diferencia clave frente a Always On AG es el alcance: FCI protege la **instancia completa** (todas las bases de datos de esa instancia, incluyendo `msdb`, logins de servidor, Agent Jobs) con una sola configuracion, mientras AG protege bases de datos individuales seleccionadas con mayor flexibilidad (puede tener replicas legibles, granularidad por base de datos), pero requiere configurar cada Availability Group por separado. Es comun combinar ambas tecnicas: FCI para alta disponibilidad local dentro de un datacenter, AG para disaster recovery hacia otra region geografica.

## Modelo mental

Pensa un Failover Cluster Instance como una sola oficina con un unico archivero fisico (el almacenamiento compartido) que puede ser operada por distintos equipos de guardia (los nodos), pero solo un equipo trabaja a la vez con el archivero completo. Si el equipo activo se ausenta, otro equipo de guardia toma el control del mismo archivero fisico inmediatamente. Esto contrasta con Always On AG, donde cada equipo (replica) tiene su propia copia completa del archivero, sincronizada constantemente.

## Ejemplo base

```sql
-- FCI se configura principalmente a nivel de infraestructura (Windows Server
-- Failover Clustering + instalacion de SQL Server como recurso clusterizado),
-- no con T-SQL directo. Verificacion del estado del cluster:

SELECT * FROM sys.dm_os_cluster_nodes;

SELECT NodeName, status_description
FROM sys.dm_os_cluster_nodes;
```

## Como se usa en un proyecto real

1. Se elige FCI cuando se necesita proteger una instancia completa (todas sus bases de datos, logins, Agent Jobs) con una unica configuracion, tipicamente para alta disponibilidad local dentro de un mismo datacenter con almacenamiento compartido disponible.
2. Se combina FCI (alta disponibilidad local) con Always On AG (disaster recovery hacia otra region) en arquitecturas criticas que requieren ambos niveles de proteccion.
3. Se evalua el costo de infraestructura de almacenamiento compartido (SAN) necesario para FCI frente a la simplicidad de AG con almacenamiento local por nodo.
4. Se conecta esto con el concepto de Failover Clustering del libro de Linux (Keepalived, Pacemaker), entendiendo que ambos ecosistemas resuelven el mismo problema de continuidad de servicio con enfoques de almacenamiento distintos.
5. Se documenta el plan de recuperacion ante fallas especifico para la arquitectura elegida (FCI, AG, o ambas combinadas).

## Criterio junior

Un junior debe entender la diferencia conceptual entre FCI (instancia completa, almacenamiento compartido) y AG (bases de datos individuales, almacenamiento replicado).

## Criterio mid

Un mid debe poder decidir cuando usar FCI versus AG segun los requisitos de un sistema especifico.

## Criterio senior

Un senior debe diseñar arquitecturas que combinen FCI y AG segun necesidades de alta disponibilidad local y disaster recovery geografico.

## Errores comunes

- Confundir FCI (protege la instancia completa via almacenamiento compartido) con AG (protege bases de datos individuales via replicacion).
- No considerar el costo y complejidad de infraestructura de almacenamiento compartido necesario para FCI.
- Asumir que FCI por si solo protege contra la perdida del almacenamiento compartido (SAN), que sigue siendo un punto unico de falla sin proteccion adicional.
- No combinar FCI local con AG para disaster recovery geografico en sistemas verdaderamente criticos.

## Practica

- [ ] Investiga los requisitos de infraestructura de un Failover Cluster Instance (WSFC, almacenamiento compartido).
- [ ] Compara el alcance de proteccion de FCI (instancia completa) versus AG (bases de datos individuales).
- [ ] Diseña una arquitectura que combine FCI local con AG para disaster recovery.
- [ ] Investiga las vistas de sistema (`sys.dm_os_cluster_nodes`) para monitorear el estado de un cluster.
- [ ] Compara este enfoque con Failover Clustering/Keepalived ya visto en el libro de Linux.

## Preguntas de repaso

- Que diferencia de alcance hay entre FCI y Always On AG?
- Por que FCI depende de almacenamiento compartido, y que riesgo introduce eso?
- En que escenario combinarias FCI con AG en una misma arquitectura?
- Como se relaciona este concepto con Failover Clustering visto en el libro de Linux?
