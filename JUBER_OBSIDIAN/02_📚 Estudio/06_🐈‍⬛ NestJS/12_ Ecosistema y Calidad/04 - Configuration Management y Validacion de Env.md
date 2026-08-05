---
tags:
  - NestJS
  - TypeScript
  - Nest-Calidad
Creado: 2026-07-27
Relacionado:
  - Ecosistema y Calidad
  - NESTJS
---

# 04 - Configuration Management y Validacion de Env

## Que vas a aprender

En este capitulo vas a profundizar en gestion de configuracion mas alla de lo basico visto en DevOps: **configuracion tipada** con clases (`class-validator` en vez de Joi), configuracion jerarquica por feature (`registerAs`), y estrategias para configuracion que cambia en runtime sin reiniciar la aplicacion (feature flags).

## Explicacion

Una alternativa a validar el schema de entorno con Joi es usar clases TypeScript decoradas con `class-validator`, aprovechando el mismo mecanismo que ya se usa para DTOs (visto en el capitulo de APIs REST), lo cual da consistencia al equipo y tipado automatico end-to-end.

`registerAs()` permite organizar la configuracion por feature (`databaseConfig`, `authConfig`, `mailConfig`) en vez de un unico objeto gigante, haciendo que cada modulo dependa solo de su porcion relevante de configuracion, inyectada con `ConfigType<typeof databaseConfig>`.

Para configuracion que necesita cambiar sin redeploy (activar una feature para un porcentaje de usuarios, un flag de mantenimiento), se necesita una fuente dinamica (base de datos, servicio de feature flags) en vez de variables de entorno estaticas leidas solo al arrancar.

## Modelo mental

Pensa la configuracion tipada como DTOs, pero para el arranque de la aplicacion en vez de para un request HTTP: mismo mecanismo de validacion, aplicado a un momento distinto del ciclo de vida. Pensa `registerAs` como cajones etiquetados en vez de un unico cajon desordenado con toda la configuracion mezclada.

## Ejemplo base

```typescript
export class EnvironmentVariables {
  @IsEnum(['development', 'production', 'test'])
  NODE_ENV: string;

  @IsString() @MinLength(32)
  JWT_SECRET: string;
}

// registerAs para configuracion por feature
export const databaseConfig = registerAs('database', () => ({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT, 10),
}));

@Injectable()
export class UsersService {
  constructor(@Inject(databaseConfig.KEY) private dbConfig: ConfigType<typeof databaseConfig>) {}
}
```

## Como se usa en un proyecto real

1. Se organiza la configuracion por feature con `registerAs`, evitando un unico objeto de configuracion gigante y poco cohesivo.
2. Se valida la configuracion con clases y `class-validator` para reusar el mismo patron mental que los DTOs de la API.
3. Para feature flags dinamicos, se usa una fuente que pueda consultarse en runtime (base de datos, servicio externo de feature flags) en vez de variables de entorno fijas.
4. Se documenta claramente la diferencia entre configuracion estatica (leida al arrancar, requiere redeploy para cambiar) y dinamica (puede cambiar sin reiniciar).
5. Se evita over-configurar: no toda decision necesita ser una variable de entorno o feature flag; a veces una constante en codigo es mas simple y clara.

## Criterio junior

Un junior debe poder definir configuracion basica organizada por feature con `registerAs` e inyectarla en un servicio.

## Criterio mid

Un mid debe validar configuracion con clases y `class-validator`, y diferenciar claramente configuracion estatica de configuracion dinamica (feature flags).

## Criterio senior

Un senior debe diseñar la estrategia de configuration management de todo el sistema, incluyendo cuando introducir un servicio de feature flags dedicado, y evitar la proliferacion descontrolada de flags que nadie recuerda para que sirven ni cuando eliminar.

## Errores comunes

- Mantener toda la configuracion en un unico objeto gigante sin organizacion por feature, dificultando saber que necesita cada modulo.
- Confundir configuracion estatica con dinamica, esperando que un cambio de variable de entorno se refleje sin reiniciar la aplicacion.
- Acumular feature flags indefinidamente sin un proceso de limpieza, generando codigo con ramas condicionales que nadie se anima a eliminar.
- No validar la configuracion, dejando pasar valores invalidos que fallan de forma confusa mas adelante en el ciclo de vida de la aplicacion.

## Practica

- [ ] Organiza la configuracion de un proyecto Nest con `registerAs` por al menos dos features distintas.
- [ ] Valida la configuracion con una clase decorada con `class-validator` en vez de un schema Joi.
- [ ] Investiga como implementarias un feature flag dinamico que no requiera redeploy.
- [ ] Audita un proyecto existente en busca de feature flags que ya no se usan y deberian eliminarse.
- [ ] Anota tu criterio para decidir cuando algo debe ser variable de entorno, feature flag, o simplemente una constante en codigo.

## Preguntas de repaso

- Que ventaja da organizar configuracion por feature con `registerAs` sobre un unico objeto gigante?
- Que diferencia hay entre configuracion estatica y configuracion dinamica (feature flags)?
- Por que es importante tener un proceso de limpieza de feature flags viejos?
- Cuando NO conviene convertir una decision en variable de entorno o feature flag?
