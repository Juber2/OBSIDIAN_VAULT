---
tags:
  - PostgreSQL
  - NestJS
  - Postgres-NestJS
Creado: 2026-07-29
Relacionado:
  - Postgres con NestJS y TypeORM
  - POSTGRESQL
---

# 01 - Conexion desde NestJS

## Que vas a aprender

En este capitulo vas a consolidar la conexion entre Postgres y NestJS, revisando con la profundidad de este libro lo ya visto de forma introductoria en el capitulo de Base de Datos del libro de NestJS: pool de conexiones, SSL, y configuracion de resiliencia.

## Explicacion

TypeORM, al conectarse a Postgres, mantiene internamente un **pool de conexiones** (configurable via `extra: { max: N }` en la configuracion de `TypeOrmModule.forRootAsync`, ya visto en el libro de NestJS), cuyo dimensionamiento debe coordinarse con `max_connections` de Postgres (ver capitulo de Arquitectura de este libro) y, si se usa PgBouncer (ver capitulo de Alta Disponibilidad), con el `default_pool_size` del pooler intermedio, evitando configurar capas de pooling incoherentes entre si.

Para conexiones a servicios gestionados en la nube (ver capitulo anterior), casi siempre se requiere **SSL** habilitado (`ssl: { rejectUnauthorized: false }` o con certificados especificos del proveedor), y es buena practica implementar reintentos de conexion con backoff (ya visto conceptualmente en el capitulo de gestion de dependencias del libro de NestJS) para tolerar interrupciones momentaneas de red sin que la aplicacion falle al arrancar.

## Modelo mental

Pensa la configuracion de conexion de TypeORM hacia Postgres como coordinar varios engranajes que deben encajar entre si: el tamaño del pool de la aplicacion, el limite de conexiones del pooler intermedio (si existe), y el `max_connections` real del motor, todos deben estar dimensionados de forma coherente, no cada uno configurado de forma aislada sin considerar a los demas.

## Ejemplo base

```typescript
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  inject: [ConfigService],
  useFactory: (config: ConfigService) => ({
    type: 'postgres',
    host: config.get('DB_HOST'),
    port: config.get<number>('DB_PORT'),
    username: config.get('DB_USER'),
    password: config.get('DB_PASSWORD'),
    database: config.get('DB_NAME'),
    entities: [__dirname + '/**/*.entity{.ts,.js}'],
    synchronize: false,
    ssl: config.get('NODE_ENV') === 'production'
      ? { rejectUnauthorized: false }
      : false,
    extra: {
      max: 20,               // pool de la aplicacion
      connectionTimeoutMillis: 5000,
    },
    retryAttempts: 5,
    retryDelay: 3000,
  }),
});
```

## Como se usa en un proyecto real

1. Se dimensiona `extra.max` del pool de TypeORM coordinado con `max_connections` de Postgres, considerando cuantas instancias de la aplicacion corren simultaneamente (cada una con su propio pool).
2. Se habilita SSL para cualquier conexion hacia un servicio gestionado en la nube, siguiendo los requisitos especificos del proveedor.
3. Se configura `retryAttempts`/`retryDelay` para tolerar interrupciones momentaneas de red al arrancar la aplicacion, evitando fallos de despliegue por problemas transitorios de conectividad.
4. Se conecta esto con PgBouncer (ver capitulo de Alta Disponibilidad de este libro) cuando la escala del sistema lo justifica, ajustando `extra.max` a un valor mas conservador dado que el pooler intermedio ya maneja la multiplexacion real.
5. Se documenta la configuracion de conexion completa como parte de los requisitos de infraestructura del proyecto, coordinando con el equipo de infraestructura si Postgres es administrado por otro equipo.

## Criterio junior

Un junior debe poder configurar una conexion basica de TypeORM hacia Postgres usando variables de entorno.

## Criterio mid

Un mid debe dimensionar el pool de conexiones de TypeORM coordinado con `max_connections` de Postgres, y habilitar SSL para conexiones a servicios gestionados.

## Criterio senior

Un senior debe diseñar la arquitectura completa de conexion entre la aplicacion y Postgres (pool de aplicacion, PgBouncer si aplica, limites del motor), documentando esa coordinacion para todo el equipo.

## Errores comunes

- No coordinar el tamaño del pool de TypeORM con `max_connections` real de Postgres, arriesgando agotar conexiones bajo carga con multiples instancias de la aplicacion corriendo simultaneamente.
- No habilitar SSL para conexiones a servicios gestionados que lo requieren, obteniendo errores de conexion confusos.
- No configurar reintentos de conexion, causando que la aplicacion falle al arrancar ante una interrupcion momentanea de red durante el despliegue.
- Configurar el pool de la aplicacion con un valor alto sin considerar que PgBouncer (si existe) ya multiplexa las conexiones reales de forma mas eficiente.

## Practica

- [ ] Configura una conexion de TypeORM hacia Postgres con pool dimensionado explicitamente.
- [ ] Habilita SSL para una conexion hacia un servicio gestionado de practica.
- [ ] Configura reintentos de conexion y simula una interrupcion momentanea de red al arrancar.
- [ ] Calcula el dimensionamiento correcto del pool considerando multiples instancias de tu aplicacion corriendo simultaneamente.
- [ ] Documenta la configuracion de conexion completa de un proyecto real tuyo.

## Preguntas de repaso

- Por que el tamaño del pool de TypeORM debe coordinarse con `max_connections` de Postgres?
- Cuando es necesario habilitar SSL en la conexion de TypeORM hacia Postgres?
- Que problema previene configurar reintentos de conexion al arrancar la aplicacion?
- Como cambia el dimensionamiento del pool de la aplicacion si se introduce PgBouncer entre medio?
