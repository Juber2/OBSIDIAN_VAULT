---
tags:
  - NestJS
  - TypeScript
  - Nest-Performance
Creado: 2026-07-27
Relacionado:
  - Performance NestJS
  - NESTJS
---

# 02 - Redis y Sesiones

## Que vas a aprender

En este capitulo vas a profundizar en **Redis** como pieza de infraestructura clave en NestJS mas alla del cache: almacenamiento de sesiones, rate limiting distribuido, locks distribuidos, y estructuras de datos utiles (sets, sorted sets) para casos especificos de negocio.

## Explicacion

Redis es una base de datos en memoria, extremadamente rapida, usada tipicamente como pieza complementaria (no reemplazo) de la base de datos principal. Ademas de cache, casos comunes en NestJS son: almacenar sesiones cuando se usa autenticacion basada en sesion en vez de JWT puro, implementar rate limiting compartido entre multiples instancias (el contador vive en Redis, no en memoria de cada instancia), locks distribuidos para evitar ejecucion concurrente de una misma tarea entre instancias (ver capitulo de Scheduling), y estructuras como sorted sets para casos como leaderboards o colas de prioridad.

## Modelo mental

Pensa Redis como una pizarra compartida y muy rapida en el medio de la oficina: cualquier instancia de tu aplicacion puede escribir o leer de ella al instante, y todas ven exactamente lo mismo, a diferencia de una nota pegada en el escritorio de cada uno (memoria local de cada instancia).

## Ejemplo base

```typescript
// Lock distribuido simple para evitar ejecucion duplicada de un cron en multiples instancias
async ejecutarConLock(key: string, fn: () => Promise<void>) {
  const acquired = await this.redis.set(key, '1', 'EX', 30, 'NX');
  if (!acquired) return; // otra instancia ya esta ejecutando esto
  try {
    await fn();
  } finally {
    await this.redis.del(key);
  }
}

// Rate limiting distribuido simple
async isAllowed(userId: string): Promise<boolean> {
  const count = await this.redis.incr(`rate:${userId}`);
  if (count === 1) await this.redis.expire(`rate:${userId}`, 60);
  return count <= 100;
}
```

## Como se usa en un proyecto real

1. Se usa Redis para todo estado que necesita compartirse entre multiples instancias de la aplicacion (sesiones, rate limiting, locks).
2. Se configuran expiraciones (`EX`) en todas las claves que representan estado temporal, para no acumular datos indefinidamente.
3. Se usa el comando `SET ... NX` (set solo si no existe) como primitiva basica para implementar locks distribuidos simples.
4. Se monitorea el uso de memoria de Redis, porque a diferencia de una base de datos en disco, un Redis mal gestionado puede quedarse sin memoria.
5. Se decide con criterio cuando algo debe vivir en Redis (rapido, temporal, compartido) versus en la base de datos principal (durable, consultable, con integridad referencial).

## Criterio junior

Un junior debe entender que Redis es una pieza distinta de la base de datos principal, y poder usarlo para un caso simple de cache o contador.

## Criterio mid

Un mid debe implementar rate limiting distribuido y locks distribuidos simples usando primitivas de Redis, sin depender de estado en memoria local de cada instancia.

## Criterio senior

Un senior debe decidir la arquitectura de uso de Redis en todo el sistema (que vive ahi, con que expiraciones, como se dimensiona y monitorea), y evaluar cuando otra estructura de datos de Redis resuelve mejor un problema especifico del dominio.

## Errores comunes

- Mantener contadores o locks en memoria local del proceso en una aplicacion con multiples instancias, rompiendo la coordinacion entre ellas.
- No poner expiracion a claves temporales en Redis, acumulando datos indefinidamente y agotando memoria.
- Usar Redis como si fuera una base de datos durable principal, sin entender que su persistencia es distinta y no reemplaza a Postgres/MySQL para datos criticos.
- No monitorear el uso de memoria de Redis, descubriendo el problema solo cuando falla en produccion.

## Practica

- [ ] Implementa un rate limiter distribuido simple usando Redis con `INCR` y `EXPIRE`.
- [ ] Implementa un lock distribuido simple usando `SET ... NX EX`.
- [ ] Configura expiracion en todas las claves temporales que uses en Redis.
- [ ] Investiga como monitorear el uso de memoria de una instancia de Redis.
- [ ] Anota que casos de tu sistema se beneficiarian de Redis mas alla de cache simple.

## Preguntas de repaso

- Por que un rate limiter en memoria local no funciona correctamente con multiples instancias?
- Que primitiva de Redis se usa tipicamente para implementar un lock distribuido simple?
- Por que es importante poner expiracion a las claves temporales en Redis?
- En que se diferencia Redis de una base de datos principal como Postgres en cuanto a durabilidad?
