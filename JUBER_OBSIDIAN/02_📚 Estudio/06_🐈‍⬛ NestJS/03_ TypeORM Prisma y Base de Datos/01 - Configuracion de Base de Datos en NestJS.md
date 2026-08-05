---
tags:
  - NestJS
  - TypeScript
  - Nest-Datos
Creado: 2026-07-27
Relacionado:
  - TypeORM Prisma y Base de Datos
  - NESTJS
---

# 01 - Configuracion de Base de Datos en NestJS

## Que vas a aprender

En este capitulo vas a aprender a configurar la conexion a base de datos en NestJS usando `TypeOrmModule.forRootAsync()`, integrando variables de entorno con `ConfigService` de forma segura y reproducible entre ambientes.

## Explicacion

NestJS no fuerza un ORM especifico: los mas usados son **TypeORM** (integracion oficial, similar a Doctrine/Eloquent) y **Prisma** (schema-first, cliente generado y tipado). La configuracion de conexion nunca debe ir hardcodeada: se arma desde variables de entorno, validadas con `ConfigModule` (ver capitulo de Ecosistema y Calidad), para que el mismo codigo funcione en local, test, staging y produccion sin cambios.

`forRootAsync()` permite inyectar `ConfigService` dentro de la factory de configuracion del ORM, evitando el problema de orden de carga entre el modulo de config y el modulo de base de datos.

## Modelo mental

Pensa la configuracion de base de datos como un contrato entre el codigo y el entorno: el codigo declara "necesito estos parametros" (host, puerto, usuario, nombre de base), y cada entorno los provee con sus propios valores, sin que el codigo fuente sepa ni le importe cuales son.

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
    synchronize: false, // nunca en produccion
  }),
});
```

## Como se usa en un proyecto real

1. Se usa `forRootAsync()` con `ConfigService` para leer credenciales desde variables de entorno.
2. `synchronize: false` siempre en cualquier ambiente que importe (staging y produccion como minimo); el esquema se maneja con migrations.
3. Se define un pool de conexiones con limites explicitos segun la capacidad del servidor de base de datos.
4. Se separan credenciales por ambiente usando archivos `.env` distintos o un vault de secretos, nunca commiteados al repo.
5. Se agrega un health check de base de datos para detectar caidas de conexion en produccion.

## Criterio junior

Un junior debe poder configurar una conexion basica a base de datos usando variables de entorno y entender por que `synchronize: true` es peligroso fuera de desarrollo local.

## Criterio mid

Un mid debe configurar pools de conexiones, manejar credenciales por ambiente de forma segura, y diagnosticar errores comunes de conexion (timeouts, limite de conexiones agotado).

## Criterio senior

Un senior debe definir la estrategia de gestion de secretos y configuracion de base de datos para todo el equipo (vault, rotacion de credenciales, limites de pool por servicio) y anticipar problemas de escalabilidad de conexiones en arquitecturas con multiples instancias.

## Errores comunes

- Dejar `synchronize: true` activo en un ambiente compartido, arriesgando perdida de datos por cambios automaticos de esquema.
- Hardcodear credenciales de base de datos en el codigo fuente.
- No limitar el tamaño del pool de conexiones y agotar conexiones disponibles en la base bajo carga.
- No tener un health check de base de datos, descubriendo caidas de conexion solo cuando un usuario reporta error.

## Practica

- [ ] Configura `TypeOrmModule.forRootAsync()` leyendo credenciales desde `ConfigService`.
- [ ] Verifica que `synchronize` este desactivado fuera de desarrollo local.
- [ ] Define un pool de conexiones con limites explicitos.
- [ ] Agrega un endpoint de health check que verifique la conexion a base de datos.
- [ ] Anota como gestionarias secretos de base de datos en produccion.

## Preguntas de repaso

- Por que `synchronize: true` es riesgoso fuera de desarrollo local?
- Que ventaja tiene `forRootAsync()` sobre `forRoot()` para leer configuracion?
- Que pasa si el pool de conexiones se agota bajo carga?
- Como verificarias en produccion que la base de datos sigue disponible?
