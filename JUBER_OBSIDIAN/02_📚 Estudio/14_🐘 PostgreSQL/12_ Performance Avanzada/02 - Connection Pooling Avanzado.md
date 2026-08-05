---
tags:
  - PostgreSQL
  - Performance
  - Postgres-Performance-Avanzada
Creado: 2026-07-29
Relacionado:
  - Performance Avanzada
  - POSTGRESQL
---

# 02 - Connection Pooling Avanzado

## Que vas a aprender

En este capitulo vas a profundizar en el dimensionamiento fino de connection pooling (ya visto con PgBouncer en el capitulo de Alta Disponibilidad), aplicando la formula practica de dimensionamiento de pools, y patrones de pooling en arquitecturas con multiples servicios.

## Explicacion

Un error comun es asumir que "mas conexiones en el pool siempre es mejor": en realidad, un pool sobredimensionado puede degradar performance, dado que Postgres tiene un numero optimo de conexiones activas simultaneas relacionado con los nucleos de CPU disponibles (mas conexiones que eso solo generan mas contencion por recursos, no mas throughput real). Una formula practica de referencia (de la documentacion de PgBouncer) es `pool_size = ((core_count * 2) + effective_spindle_count)`, donde `effective_spindle_count` considera el tipo de almacenamiento (mayor para SSD/NVMe rapidos).

En arquitecturas con **multiples servicios** (microservicios, ver capitulo de Arquitectura del libro de NestJS) conectandose a la misma base de datos, cada servicio con su propio pool de aplicacion puede sumar a un total que excede `max_connections` real, incluso con PgBouncer intermedio si no esta bien dimensionado. La solucion es centralizar el dimensionamiento pensando en el **total agregado** de todos los servicios, no cada uno de forma aislada.

## Modelo mental

Pensa el dimensionamiento de pools como calcular cuantos cajeros necesita un banco: no es "cuantos mas cajeros, mejor" indefinidamente, hay un punto optimo relacionado con cuantas transacciones puede procesar el sistema de fondo (los nucleos de CPU) simultaneamente. Pasado ese punto, agregar mas cajeros solo genera mas gente esperando en fila para acceder al mismo sistema limitado de fondo, sin mejorar el throughput real.

## Ejemplo base

```
Formula practica de dimensionamiento (referencia de PgBouncer):
pool_size = (nucleos_cpu * 2) + spindle_count

Para un servidor de 8 nucleos con SSD rapido (spindle_count ~ 4):
pool_size = (8 * 2) + 4 = 20 conexiones activas optimas
```

```sql
-- Verificar conexiones activas reales vs el limite configurado
SELECT count(*) AS conexiones_activas FROM pg_stat_activity WHERE state = 'active';
SHOW max_connections;
```

## Como se usa en un proyecto real

1. Se calcula el tamaño del pool usando la formula practica basada en nucleos de CPU disponibles, en vez de un numero arbitrario "razonable".
2. Se piensa el dimensionamiento en terminos del **total agregado** de todos los servicios que se conectan a la misma base de datos, no cada uno de forma aislada.
3. Se monitorea `pg_stat_activity` regularmente para verificar cuantas conexiones estan realmente activas versus el limite configurado, ajustando segun evidencia real.
4. Se usa PgBouncer (ver capitulo de Alta Disponibilidad) como capa de multiplexacion que permite que muchos clientes de aplicacion compartan un pool real mas pequeño y correctamente dimensionado hacia Postgres.
5. Se conecta esto con el capitulo de arquitectura de microservicios del libro de NestJS, donde el dimensionamiento de pooling se vuelve una decision de infraestructura compartida entre servicios.

## Criterio junior

Un junior debe entender que un pool mas grande no siempre significa mejor performance.

## Criterio mid

Un mid debe aplicar la formula practica de dimensionamiento de pool basada en nucleos de CPU disponibles.

## Criterio senior

Un senior debe diseñar el dimensionamiento de pooling agregado de un sistema con multiples servicios, coordinando PgBouncer y los pools de cada aplicacion individual.

## Errores comunes

- Asumir que aumentar el tamaño del pool siempre mejora el throughput, sin considerar el punto de contencion real relacionado con nucleos de CPU.
- Dimensionar el pool de cada servicio de forma aislada, sin considerar el total agregado que todos suman hacia la misma base de datos.
- No monitorear conexiones activas reales, dimensionando pools basado en suposiciones en vez de evidencia medida.
- No usar PgBouncer como capa de multiplexacion en arquitecturas con muchos servicios cliente, dejando que cada uno abra demasiadas conexiones directas.

## Practica

- [ ] Calcula el tamaño de pool optimo usando la formula practica para un servidor hipotetico con una cantidad especifica de nucleos.
- [ ] Monitorea `pg_stat_activity` de un sistema real o de practica y compara con el limite configurado.
- [ ] Diseña el dimensionamiento agregado de pools para una arquitectura hipotetica con tres microservicios conectados a la misma base de datos.
- [ ] Investiga como PgBouncer permite que muchos clientes compartan un pool real mas pequeño.
- [ ] Ajusta el pool de un proyecto real basandote en la formula practica en vez de un valor arbitrario.

## Preguntas de repaso

- Por que un pool de conexiones sobredimensionado puede degradar performance en vez de mejorarla?
- Que factores considera la formula practica de dimensionamiento de pool de PgBouncer?
- Por que el dimensionamiento debe pensarse en terminos del total agregado en arquitecturas de microservicios?
- Como monitorearias si el tamaño de pool actual es apropiado para la carga real?
