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

# 02 - Replicacion Logica

## Que vas a aprender

En este capitulo vas a aprender **replicacion logica** (`publication`/`subscription`), una alternativa a Streaming Replication que replica a nivel de **cambios de datos** en vez de bloques fisicos de WAL, permitiendo casos de uso que la replicacion fisica no puede resolver.

## Explicacion

Mientras Streaming Replication (tema anterior) replica bytes fisicos del WAL (exigiendo que la replica sea una copia identica byte a byte del primario, misma version de Postgres), la **replicacion logica** transmite los cambios de datos decodificados (inserts, updates, deletes especificos), permitiendo replicar **solo tablas seleccionadas** (no la base de datos completa), entre **versiones distintas** de Postgres, o incluso hacia motores de bases de datos distintos con herramientas de terceros que consumen ese flujo logico.

Se configura con una **publicacion** (`CREATE PUBLICATION`) en el origen, que define que tablas se replican, y una **suscripcion** (`CREATE SUBSCRIPTION`) en el destino, que se conecta a esa publicacion y aplica los cambios. Casos de uso tipicos: migrar datos entre versiones de Postgres con downtime minimo, sincronizar un subconjunto de tablas hacia un sistema de analytics separado, o alimentar un pipeline de datos en tiempo real sin necesitar replicar la base completa.

## Modelo mental

Pensa Streaming Replication como fotocopiar exactamente cada pagina de un libro tal cual esta impresa, necesitando el mismo formato de papel e imprenta. Replicacion logica es como tener un dictado en vivo del contenido: alguien lee en voz alta que cambio especificamente ("se agrego esta fila, se modifico esta otra"), permitiendo que el receptor lo transcriba en un formato distinto, en un idioma distinto (version distinta de Postgres), o solo tome nota de los capitulos que le interesan (tablas seleccionadas).

## Ejemplo base

```sql
-- En el servidor origen
CREATE PUBLICATION pub_pedidos FOR TABLE pedidos, pedido_items;

-- En el servidor destino (puede ser otra version de Postgres, u otra base especifica)
CREATE SUBSCRIPTION sub_pedidos
CONNECTION 'host=origen.interno dbname=mi_app user=replicador password=...'
PUBLICATION pub_pedidos;

-- Monitorear el estado de la suscripcion
SELECT * FROM pg_stat_subscription;
```

## Como se usa en un proyecto real

1. Se usa replicacion logica para migrar entre versiones mayores de Postgres con downtime minimo: se configura la suscripcion, se espera a que este sincronizada, y se corta el trafico al nuevo servidor casi instantaneamente.
2. Se usa para sincronizar solo las tablas necesarias hacia un sistema de analytics o reportes separado, sin replicar datos sensibles o innecesarios para ese proposito.
3. Se evalua replicacion logica cuando Streaming Replication es demasiado rigida para el caso de uso (necesidad de replicar solo un subconjunto de tablas, o entre versiones distintas).
4. Se monitorea `pg_stat_subscription` para verificar que la suscripcion sigue sincronizada y no acumula retraso significativo.
5. Se documenta claramente el proposito de cada publicacion/suscripcion activa, dado que puede no ser evidente para quien administra el sistema despues.

## Criterio junior

Un junior debe entender la diferencia conceptual entre replicacion fisica (Streaming) y logica, y poder configurar una publicacion/suscripcion basica.

## Criterio mid

Un mid debe usar replicacion logica para sincronizar tablas seleccionadas hacia un destino especifico, monitoreando su estado.

## Criterio senior

Un senior debe diseñar una migracion de version mayor de Postgres usando replicacion logica para minimizar downtime, y decidir cuando replicacion logica es la herramienta correcta frente a Streaming Replication.

## Errores comunes

- Usar Streaming Replication cuando en realidad se necesitaba replicar solo un subconjunto de tablas, forzando una replicacion completa innecesaria.
- No monitorear `pg_stat_subscription`, perdiendo visibilidad de si una suscripcion se desincronizo silenciosamente.
- No documentar el proposito de publicaciones/suscripciones activas, generando confusion sobre por que existen ciertos flujos de replicacion en el sistema.
- Asumir que replicacion logica replica automaticamente cambios de esquema (DDL), cuando en realidad requiere manejo manual o coordinado de esos cambios.

## Practica

- [ ] Configura una publicacion para tablas seleccionadas en un servidor de practica.
- [ ] Configura una suscripcion en otro servidor (o base de datos) que consuma esa publicacion.
- [ ] Verifica que los cambios se replican correctamente y monitorea `pg_stat_subscription`.
- [ ] Investiga como usarias replicacion logica para una migracion de version mayor con downtime minimo.
- [ ] Documenta un caso de uso real donde replicacion logica seria mejor que Streaming Replication para tu proyecto.

## Preguntas de repaso

- Que diferencia fundamental hay entre replicacion fisica (Streaming) y logica?
- Que caso de uso permite replicacion logica que Streaming Replication no puede resolver?
- Como usarias replicacion logica para minimizar downtime en una migracion de version mayor?
- Que limitacion tiene replicacion logica respecto a cambios de esquema (DDL)?
