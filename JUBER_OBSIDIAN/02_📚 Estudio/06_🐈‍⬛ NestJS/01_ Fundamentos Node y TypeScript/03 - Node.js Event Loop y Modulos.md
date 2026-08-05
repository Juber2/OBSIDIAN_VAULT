---
tags:
  - NestJS
  - TypeScript
  - Nest-Fundamentos
Creado: 2026-07-27
Relacionado:
  - Fundamentos Node y TypeScript
  - NESTJS
---

# 03 - Node.js Event Loop y Modulos

## Que vas a aprender

En este capitulo vas a repasar como funciona el **Event Loop de Node.js**, por que Node es single-threaded pero no bloqueante, y como eso afecta directamente el diseño de servicios NestJS (que operaciones bloquean, cuales no, y como evitar bloquear el hilo principal).

## Explicacion

Node.js ejecuta JavaScript en un solo hilo, pero delega operaciones de I/O (red, disco, DB) al sistema operativo o a un thread pool (libuv), liberando el hilo principal mientras esperan. El Event Loop es el mecanismo que va procesando callbacks, promesas y timers a medida que esas operaciones terminan.

Para NestJS esto importa porque un endpoint que hace trabajo **CPU-bound** pesado (por ejemplo, procesar una imagen o un calculo intensivo de forma sincrona) bloquea el Event Loop y afecta a **todas** las requests concurrentes, no solo a la que lo pidio. La solucion tipica es mover ese trabajo a un worker thread, a un proceso separado o a una cola (ver Colas y Eventos).

## Modelo mental

Pensa el Event Loop como un mesero con una sola mano: puede atender muchas mesas rapido si solo tiene que anotar pedidos y pasarlos a la cocina (I/O asincrono), pero si se pone a cocinar el mismo un plato (trabajo sincrono pesado), todas las demas mesas esperan.

## Ejemplo base

```typescript
// Mal: bloquea el Event Loop
@Get('reporte-pesado')
generarReporte() {
  let total = 0;
  for (let i = 0; i < 5_000_000_000; i++) total += i; // bloquea todo
  return { total };
}

// Mejor: delegar a un worker thread o a una cola de jobs
@Get('reporte-pesado')
async generarReporte() {
  return this.reportesQueue.add('generar', {});
}
```

## Como se usa en un proyecto real

1. Se identifican los endpoints que hacen trabajo CPU-bound (reportes, procesamiento de imagenes, calculos).
2. Ese trabajo se mueve a una cola (BullMQ) o a un `worker_threads` si debe resolverse en el mismo proceso.
3. Las operaciones de I/O (DB, HTTP externo, filesystem) siempre se usan con su version async/await.
4. Se monitorea el "event loop lag" en produccion como metrica de salud.
5. Se evita usar versiones sincronas de APIs de Node (`fs.readFileSync`, etc.) en el camino caliente de un request.

## Criterio junior

Un junior debe entender que Node es single-threaded, que async/await no crea hilos nuevos, y evitar operaciones sincronas obvias dentro de un controller.

## Criterio mid

Un mid debe poder identificar codigo que bloquea el Event Loop y proponer una solucion (worker thread, cola, proceso separado) segun el caso.

## Criterio senior

Un senior debe poder diseñar la estrategia general de la aplicacion para trabajo pesado (que va sincrono, que va a cola, que va a un microservicio dedicado) y definir metricas de event loop lag como parte de la observabilidad.

## Errores comunes

- Usar bucles sincronos pesados dentro de un endpoint HTTP.
- Confundir "asincrono" con "en paralelo" cuando en realidad sigue siendo un solo hilo.
- Usar APIs sincronas de Node (`*Sync`) en produccion sin justificacion.
- No monitorear el event loop lag hasta que aparece un incidente de latencia.

## Practica

- [ ] Escribe un endpoint que bloquee el Event Loop a proposito y mide la latencia de otro endpoint en paralelo.
- [ ] Mueve ese trabajo a un worker thread o a una cola.
- [ ] Compara la latencia antes y despues.
- [ ] Investiga como medir event loop lag en tu stack de monitoreo.
- [ ] Anota una regla de equipo: "que trabajo nunca va directo en un controller".

## Preguntas de repaso

- Por que Node.js puede manejar miles de conexiones con un solo hilo?
- Que pasa si un endpoint ejecuta un calculo CPU-bound de forma sincrona?
- Que alternativas existen para trabajo pesado en NestJS?
- Como detectarias en produccion que el Event Loop se esta bloqueando?
