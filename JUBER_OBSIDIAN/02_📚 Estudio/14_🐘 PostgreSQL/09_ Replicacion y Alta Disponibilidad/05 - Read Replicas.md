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

# 05 - Read Replicas

## Que vas a aprender

En este capitulo vas a cerrar el capitulo consolidando el uso practico de **read replicas**: como distribuir carga de lectura usando las replicas de Streaming Replication ya vistas, y el problema de **replication lag** que hay que manejar conscientemente en la aplicacion.

## Explicacion

Una **read replica** es simplemente una replica de Streaming Replication (ver primer tema del capitulo) usada activamente para servir consultas de **solo lectura**, distribuyendo la carga de lectura del primario hacia una o mas replicas, mientras todas las escrituras siguen yendo exclusivamente al primario. Esto escala la capacidad de lectura de un sistema (relevante para reportes pesados, dashboards, o simplemente alto volumen de trafico de lectura) sin necesitar escalar verticalmente el primario.

El desafio real es el **replication lag**: como la replicacion es asincrona (en el caso mas comun), existe un pequeño desfase entre cuando un dato se escribe en el primario y cuando aparece disponible en la replica. Si una aplicacion escribe un dato y **inmediatamente** lo lee desde una replica, puede no encontrarlo todavia (un problema de consistencia eventual). La solucion tipica es dirigir a la replica solo lecturas que **toleran** ese desfase (reportes, listados generales), y forzar lecturas criticas post-escritura (como "mostrar el pedido recien creado") hacia el primario.

## Modelo mental

Pensa una read replica como una sucursal de una biblioteca que recibe copias de los libros nuevos poco despues de que la biblioteca central los adquiere, con un pequeño desfase de tiempo. Para consultas generales (buscar cualquier libro popular), la sucursal funciona perfectamente y alivia la carga de la central. Pero si alguien acaba de donar un libro a la biblioteca central y quiere verificar inmediatamente que se registro, tiene que consultar directamente en la central, porque la sucursal todavia podria no tener la copia actualizada.

## Ejemplo base

```typescript
// Configuracion conceptual en TypeORM con multiples fuentes de datos (primario + replica)
const primarioSource = new DataSource({ host: 'primario.interno', /* ... */ });
const replicaSource = new DataSource({ host: 'replica.interno', /* ... */ });

// Escrituras y lecturas criticas post-escritura: siempre al primario
await primarioSource.getRepository(Pedido).save(nuevoPedido);

// Lecturas que toleran cierto desfase: hacia la replica
const reporteVentas = await replicaSource.getRepository(Pedido)
  .createQueryBuilder('p')
  .select('SUM(p.total)', 'total')
  .getRawOne();
```

```sql
-- Verificar el retraso actual de una replica (en segundos)
SELECT EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp())) AS lag_segundos;
```

## Como se usa en un proyecto real

1. Se dirigen consultas de reportes, dashboards, y listados generales hacia read replicas, liberando capacidad del primario para operaciones criticas de escritura.
2. Se dirigen explicitamente al primario las lecturas que necesitan reflejar inmediatamente una escritura reciente (como confirmar la creacion de un recurso justo despues de crearlo).
3. Se monitorea el replication lag activamente, alertando si crece mas alla de un umbral aceptable para el negocio.
4. Se documenta claramente en el codigo que consultas van a replica versus primario, evitando que decisiones de enrutamiento queden implicitas y confusas.
5. Se conecta esto con el capitulo de Escalabilidad del libro de NestJS, donde el mismo principio de escalar lecturas independientemente de escrituras ya se discutio a nivel de aplicacion.

## Criterio junior

Un junior debe entender que es una read replica y el concepto basico de replication lag.

## Criterio mid

Un mid debe decidir que consultas de una aplicacion pueden dirigirse a una replica versus cuales necesitan el primario por consistencia inmediata.

## Criterio senior

Un senior debe diseñar la estrategia completa de distribucion de lecturas entre primario y replicas para un sistema de alto trafico, monitoreando lag activamente y estableciendo umbrales de alerta.

## Errores comunes

- Dirigir una lectura critica post-escritura hacia una replica, mostrando al usuario que su accion "no se guardo" cuando en realidad solo no habia llegado todavia por el lag de replicacion.
- No monitorear el replication lag, dejando que crezca sin control hasta afectar significativamente la experiencia de usuarios que leen desde replicas.
- No documentar claramente en el codigo que consultas van a replica versus primario, generando confusion sobre el comportamiento esperado.
- Dirigir toda la carga de lectura al primario sin aprovechar replicas disponibles, desperdiciando capacidad de escalado ya invertida en infraestructura.

## Practica

- [ ] Configura una read replica de practica (o reutiliza la del primer tema del capitulo).
- [ ] Simula una escritura en el primario y mide cuanto tarda en aparecer en la replica.
- [ ] Diseña que consultas de un proyecto real dirigirias a replica versus primario.
- [ ] Consulta el lag actual de una replica con `pg_last_xact_replay_timestamp()`.
- [ ] Documenta la estrategia de enrutamiento de lecturas que aplicarias en un sistema de alto trafico.

## Preguntas de repaso

- Que es una read replica y que tipo de carga esta diseñada para aliviar?
- Que es el replication lag y por que puede causar un problema de consistencia percibida por el usuario?
- Que tipo de lectura deberia dirigirse siempre al primario, nunca a una replica?
- Como monitorearias el replication lag de un sistema en produccion?
