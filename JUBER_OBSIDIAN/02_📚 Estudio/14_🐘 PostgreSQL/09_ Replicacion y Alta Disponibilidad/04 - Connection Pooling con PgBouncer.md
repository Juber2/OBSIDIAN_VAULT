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

# 04 - Connection Pooling con PgBouncer

## Que vas a aprender

En este capitulo vas a aprender **PgBouncer**, el pooler de conexiones estandar para Postgres, resolviendo directamente el problema de overhead de conexiones ya identificado en el capitulo de Arquitectura (cada conexion es un proceso completo del sistema operativo).

## Explicacion

Como cada conexion a Postgres es un proceso del sistema operativo (ver capitulo de Arquitectura), abrir y cerrar conexiones frecuentemente (como haria una aplicacion NestJS sin pooling, creando una conexion nueva por cada request) es costoso. **PgBouncer** es un proxy liviano que mantiene un **pool** de conexiones reales ya abiertas hacia Postgres, y las reasigna eficientemente entre las conexiones que la aplicacion solicita, evitando el costo de crear un proceso backend nuevo en cada ocasion.

PgBouncer soporta tres modos de pooling: **session** (una conexion del pool se dedica a un cliente durante toda su sesion, el mas compatible pero menos eficiente), **transaction** (una conexion del pool se asigna solo durante una transaccion, liberandose inmediatamente despues, el mas usado en produccion por su eficiencia), y **statement** (se libera despues de cada sentencia individual, el mas agresivo pero incompatible con transacciones multi-sentencia). El modo transaccion es el default recomendado para la mayoria de aplicaciones web con NestJS/TypeORM.

## Modelo mental

Pensa PgBouncer como el sistema de taxis compartidos de una ciudad: en vez de que cada pasajero (conexion de aplicacion) tenga que esperar a que se fabrique un auto nuevo (proceso backend de Postgres) para su viaje, hay una flota ya circulando (pool de conexiones reales) que se reasigna eficientemente entre pasajeros a medida que van llegando y bajando, muchisimo mas rapido que fabricar un auto por cada viaje individual.

## Ejemplo base

```ini
# pgbouncer.ini
[databases]
mi_app = host=postgres.interno port=5432 dbname=mi_app

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
```

```typescript
// La aplicacion NestJS se conecta a PgBouncer, no directamente a Postgres
TypeOrmModule.forRoot({
  host: 'pgbouncer.interno',   // en vez de conectarse directo a postgres.interno
  port: 6432,                   // puerto tipico de PgBouncer
  // ... resto de configuracion
});
```

## Como se usa en un proyecto real

1. Se coloca PgBouncer entre la aplicacion NestJS y Postgres en cualquier sistema de produccion, evitando que la aplicacion abra conexiones directas sin control.
2. Se usa modo `transaction` como default, dado que balancea bien eficiencia con compatibilidad para la mayoria de patrones de uso de TypeORM.
3. Se dimensiona `default_pool_size` segun la capacidad real de conexiones del servidor Postgres (ver `max_connections` en el capitulo de Arquitectura), evitando agotar conexiones reales del motor.
4. Se verifica compatibilidad de features especificas de Postgres (como `LISTEN`/`NOTIFY`, o ciertas funciones de sesion) con el modo `transaction`, dado que algunas no funcionan correctamente en ese modo.
5. Se conecta esto directamente con el capitulo de Arquitectura de este mismo libro, cerrando el circulo entre "por que cada conexion es cara" y "como se resuelve eficientemente en la practica".

## Criterio junior

Un junior debe entender por que PgBouncer reduce el overhead de conexiones, y poder configurar una conexion basica a traves de el.

## Criterio mid

Un mid debe elegir el modo de pooling apropiado (`transaction` como default) y dimensionar `default_pool_size` segun la capacidad real del servidor.

## Criterio senior

Un senior debe diseñar la arquitectura de connection pooling de un sistema completo, considerando compatibilidad de features especificas con cada modo, y coordinar con el dimensionamiento de `max_connections` de Postgres.

## Errores comunes

- No usar ningun pooler de conexiones, dejando que la aplicacion abra conexiones directas sin control, agotando `max_connections` bajo carga.
- Usar modo `session` cuando `transaction` seria mas eficiente para el patron real de uso de la aplicacion.
- Dimensionar `default_pool_size` sin considerar la capacidad real de `max_connections` del servidor Postgres detras del pooler.
- Usar features de Postgres incompatibles con modo `transaction` (como ciertas variables de sesion) sin verificar esa compatibilidad primero.

## Practica

- [ ] Configura PgBouncer delante de una instancia Postgres de practica.
- [ ] Conecta una aplicacion (o `psql`) a traves de PgBouncer en vez de directo a Postgres.
- [ ] Compara el modo `session` con `transaction` para el mismo patron de conexiones.
- [ ] Dimensiona `default_pool_size` considerando un `max_connections` hipotetico del servidor.
- [ ] Investiga que features de Postgres podrian ser incompatibles con modo `transaction` en tu proyecto.

## Preguntas de repaso

- Por que PgBouncer reduce significativamente el overhead de conexiones frente a conectar directamente a Postgres?
- Que diferencia hay entre los modos `session`, `transaction` y `statement` de PgBouncer?
- Por que modo `transaction` es el default recomendado para la mayoria de aplicaciones web?
- Como dimensionarias `default_pool_size` en relacion a `max_connections` del servidor Postgres real?
