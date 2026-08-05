---
tags:
  - PostgreSQL
  - Proyectos
  - Postgres-Proyectos
Creado: 2026-07-29
Relacionado:
  - Proyectos
  - POSTGRESQL
---

# 03 - Proyecto Replicacion Basica

## Que vas a construir

Un cluster Postgres primario-replica basico usando Streaming Replication, con PgBouncer delante, aplicando de punta a punta el capitulo de Replicacion y Alta Disponibilidad, en containers Docker (conectando con ese libro).

## Alcance funcional

- Un primario y al menos una replica en Streaming Replication asincrona, corriendo en containers Docker separados.
- Verificacion del estado de replicacion con `pg_stat_replication`.
- PgBouncer configurado delante del primario, en modo `transaction`.
- Una prueba de replication lag: escribir en el primario y medir cuanto tarda en aparecer en la replica.
- Documentacion de que consultas dirigirias a la replica versus al primario en un sistema real.

## Checklist tecnica

- [ ] La replica se sincroniza correctamente con el primario, verificado con `pg_stat_replication`.
- [ ] Un cambio escrito en el primario aparece en la replica en un tiempo razonable, medido explicitamente.
- [ ] PgBouncer permite conexiones a traves de si mismo hacia el primario correctamente.
- [ ] Se documenta el criterio de que consultas van a replica versus primario para un caso de uso real.
- [ ] Toda la configuracion (Streaming Replication, PgBouncer) esta declarada en `docker-compose.yml` versionado.

## Criterio de exito

El proyecto se considera completo cuando el cluster completo (primario + replica + PgBouncer) levanta desde cero con `docker compose up`, la replicacion funciona verificablemente, y existe documentacion clara de la estrategia de enrutamiento de lecturas para un sistema real.

## Errores comunes a evitar

- No verificar realmente el estado de la replicacion, asumiendo que "deberia estar funcionando" sin confirmarlo con `pg_stat_replication`.
- Dirigir lecturas criticas post-escritura hacia la replica sin considerar el replication lag medido.
- No versionar la configuracion completa del cluster en `docker-compose.yml`.

## Extension opcional

Investigar (sin necesariamente implementar completo) como Patroni automatizaria el failover de este mismo cluster, documentando el diseño conceptual.
