---
tags:
  - NestJS
  - TypeScript
  - Nest-DevOps
Creado: 2026-07-27
Relacionado:
  - DevOps Deploy
  - NESTJS
---

# 01 - Ambientes y ConfigModule

## Que vas a aprender

En este capitulo vas a aprender a manejar configuracion por ambiente en NestJS con `@nestjs/config`: carga de archivos `.env`, validacion de variables de entorno con schemas, y como evitar que la aplicacion arranque con configuracion incompleta o invalida.

## Explicacion

`ConfigModule.forRoot()` carga variables de entorno desde archivos `.env` (distintos por ambiente: `.env.development`, `.env.production`) y las expone via `ConfigService` de forma inyectable, evitando el uso disperso de `process.env` en todo el codigo. Mas importante aun: permite definir un **schema de validacion** (con `Joi` o `class-validator`) que verifica, al arrancar la aplicacion, que todas las variables requeridas esten presentes y con el formato correcto.

Esto convierte errores de configuracion (una variable faltante, un puerto no numerico) de bugs silenciosos que aparecen en produccion horas despues, en fallos de arranque inmediatos y claros.

## Modelo mental

Pensa la validacion de configuracion como un checklist de despegue de un avion: si falta algo critico, el avion no despega. Es mucho mejor descubrir un problema de configuracion en el momento de arrancar la aplicacion que descubrirlo cuando un usuario real hace una request y algo falla a mitad de camino.

## Ejemplo base

```typescript
const envSchema = Joi.object({
  NODE_ENV: Joi.string().valid('development', 'production', 'test').required(),
  DB_HOST: Joi.string().required(),
  DB_PORT: Joi.number().default(5432),
  JWT_SECRET: Joi.string().min(32).required(),
});

ConfigModule.forRoot({
  isGlobal: true,
  validationSchema: envSchema,
  envFilePath: `.env.${process.env.NODE_ENV}`,
});
```

## Como se usa en un proyecto real

1. Se define un schema de validacion completo de todas las variables de entorno que la aplicacion necesita, sin excepciones.
2. `ConfigModule` se registra como global (`isGlobal: true`) para poder inyectar `ConfigService` en cualquier modulo sin reimportarlo.
3. Nunca se accede a `process.env` directamente en el codigo de negocio; siempre se pasa por `ConfigService`, facilitando testing y mockeo.
4. Se mantienen archivos `.env.example` (sin valores reales) commiteados al repo, documentando que variables existen sin exponer secretos.
5. Los secretos reales nunca se commitean; se gestionan via vault, variables de entorno del sistema de deploy, o secretos de la plataforma cloud.

## Criterio junior

Un junior debe poder configurar `ConfigModule` basico y usar `ConfigService` para leer una variable de entorno en un servicio.

## Criterio mid

Un mid debe definir un schema de validacion completo para las variables criticas del proyecto, y asegurar que la aplicacion falle rapido y claro si falta configuracion.

## Criterio senior

Un senior debe definir la estrategia de gestion de configuracion y secretos para todos los ambientes del equipo (local, staging, produccion), incluyendo rotacion de secretos y separacion de responsabilidades sobre quien puede ver que configuracion.

## Errores comunes

- Usar `process.env` directo disperso en el codigo, dificultando testing y sin validacion centralizada.
- No validar variables de entorno, dejando que la aplicacion arranque "a medias" y falle de forma confusa mas adelante.
- Commitear archivos `.env` con secretos reales al repositorio.
- No diferenciar configuracion por ambiente, arriesgando que configuracion de desarrollo llegue a produccion por error.

## Practica

- [ ] Configura `ConfigModule` con un schema de validacion para las variables criticas de un proyecto de practica.
- [ ] Provoca a proposito una variable faltante y verifica que la aplicacion falle al arrancar con un mensaje claro.
- [ ] Reemplaza cualquier uso directo de `process.env` por `ConfigService`.
- [ ] Crea un archivo `.env.example` documentando las variables necesarias sin exponer valores reales.
- [ ] Anota tu estrategia de gestion de secretos para produccion.

## Preguntas de repaso

- Que ventaja da validar variables de entorno al arrancar la aplicacion en vez de descubrir el problema en produccion?
- Por que conviene usar `ConfigService` en vez de `process.env` directo?
- Que deberia contener un archivo `.env.example` y que no?
- Como gestionarias secretos reales sin commitearlos al repositorio?
