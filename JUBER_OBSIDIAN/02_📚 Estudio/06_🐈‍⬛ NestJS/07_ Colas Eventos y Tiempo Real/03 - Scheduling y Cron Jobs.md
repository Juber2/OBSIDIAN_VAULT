---
tags:
  - NestJS
  - TypeScript
  - Nest-Async
Creado: 2026-07-27
Relacionado:
  - Colas Eventos y Tiempo Real
  - NESTJS
---

# 03 - Scheduling y Cron Jobs

## Que vas a aprender

En este capitulo vas a aprender a programar tareas recurrentes en NestJS con `@nestjs/schedule`: cron jobs, intervalos y timeouts declarativos, y los riesgos de correr scheduling en aplicaciones con multiples instancias.

## Explicacion

`@nestjs/schedule` permite declarar tareas programadas directamente como metodos decorados (`@Cron()`, `@Interval()`, `@Timeout()`), sin necesitar un cron externo del sistema operativo. Es util para tareas como limpiar datos expirados, enviar resumenes periodicos, o sincronizar datos con un sistema externo.

El riesgo principal aparece cuando la aplicacion corre en **multiples instancias** (por escalabilidad horizontal): si cada instancia tiene su propio scheduler activo, la misma tarea se ejecutara multiples veces en paralelo, lo cual puede ser incorrecto o directamente peligroso (por ejemplo, enviar el mismo email de resumen varias veces).

## Modelo mental

Pensa el scheduling como una alarma que cada instancia de tu aplicacion trae puesta de fabrica: si tenes cinco instancias corriendo, tenes cinco alarmas sonando a la misma hora, no una sola. Hay que decidir explicitamente quien "apaga" las alarmas de mas.

## Ejemplo base

```typescript
@Injectable()
export class LimpiezaService {
  @Cron('0 3 * * *') // todos los dias a las 3am
  async limpiarTokensExpirados() {
    await this.tokensRepo.delete({ expiresAt: LessThan(new Date()) });
  }

  @Interval(60000)
  async chequearSaludExterna() {
    // se ejecuta cada 60 segundos
  }
}
```

## Como se usa en un proyecto real

1. Se identifica que tareas realmente necesitan ejecutarse en un horario o intervalo fijo.
2. Si la aplicacion corre en multiples instancias, se usa un mecanismo de exclusion (lock distribuido en Redis, o delegar el scheduling a un solo worker dedicado) para evitar ejecucion duplicada.
3. Se loggea cada ejecucion de tarea programada con su resultado, para poder auditar si corrio correctamente.
4. Se definen timeouts y manejo de errores dentro de cada tarea, para que un fallo no deje el cron en un estado raro.
5. Para tareas pesadas, se prefiere que el cron solo encole un job en BullMQ, delegando el trabajo real al worker.

## Criterio junior

Un junior debe poder declarar una tarea programada simple con `@Cron()` y entender la sintaxis basica de expresiones cron.

## Criterio mid

Un mid debe reconocer el riesgo de ejecucion duplicada en multiples instancias y proponer una solucion (lock distribuido, instancia dedicada al scheduler).

## Criterio senior

Un senior debe diseñar la estrategia de scheduling de todo el sistema en un entorno de multiples instancias, decidir que tareas ameritan un lock distribuido, y auditar el impacto de tareas programadas pesadas sobre la base de datos en horarios de uso real.

## Errores comunes

- Correr `@nestjs/schedule` sin ningun mecanismo de exclusion en una aplicacion desplegada con multiples instancias, duplicando efectos.
- No loggear la ejecucion de tareas programadas, sin forma de saber si corrieron o fallaron.
- Poner trabajo pesado directamente dentro del cron en vez de delegarlo a una cola, bloqueando el Event Loop de esa instancia.
- Programar tareas pesadas en horarios de trafico alto, compitiendo con requests de usuarios reales por recursos de base de datos.

## Practica

- [ ] Declara una tarea programada simple con `@Cron()`.
- [ ] Investiga como implementar un lock distribuido con Redis para evitar ejecucion duplicada.
- [ ] Agrega logging a una tarea programada para auditar su ejecucion.
- [ ] Refactoriza una tarea pesada para que solo encole un job en vez de ejecutar el trabajo directo.
- [ ] Anota en que horario deberian correr las tareas pesadas de tu sistema.

## Preguntas de repaso

- Que problema aparece al correr scheduling en multiples instancias de la misma aplicacion?
- Que estrategias existen para evitar ejecucion duplicada de una tarea programada?
- Por que conviene que un cron solo encole un job en vez de hacer el trabajo pesado directamente?
- Como auditarias si una tarea programada corrio correctamente anoche?
