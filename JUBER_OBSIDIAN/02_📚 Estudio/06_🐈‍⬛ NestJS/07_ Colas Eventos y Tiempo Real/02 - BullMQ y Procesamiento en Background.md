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

# 02 - BullMQ y Procesamiento en Background

## Que vas a aprender

En este capitulo vas a aprender a usar **BullMQ** (con `@nestjs/bullmq`) para procesar trabajo pesado o de larga duracion en background, respaldado por Redis, con reintentos y persistencia real.

## Explicacion

A diferencia de `EventEmitter` (en memoria, se pierde si el proceso se cae), BullMQ persiste los jobs en Redis: si el servidor se reinicia a mitad de un job, el job sigue en la cola y puede reintentarse. Esto lo hace apto para trabajo que **debe** ejecutarse eventualmente: enviar emails transaccionales, procesar pagos, generar reportes pesados, redimensionar imagenes.

Un **productor** agrega jobs a la cola (`queue.add()`), y uno o mas **workers** (`@Processor()`) los consumen, potencialmente en un proceso separado del servidor HTTP para no competir por CPU con las requests entrantes.

## Modelo mental

Pensa BullMQ como una casilla de correo persistente: el productor deja una carta con instrucciones, y aunque la oficina de correos (el servidor) se apague y prenda de nuevo, la carta sigue ahi esperando a que alguien (el worker) la procese, con reintentos automaticos si algo sale mal.

## Ejemplo base

```typescript
@Injectable()
export class ReportesService {
  constructor(@InjectQueue('reportes') private queue: Queue) {}

  async solicitarReporte(userId: string) {
    await this.queue.add('generar', { userId }, { attempts: 3, backoff: 5000 });
  }
}

@Processor('reportes')
export class ReportesProcessor extends WorkerHost {
  async process(job: Job<{ userId: string }>) {
    // trabajo pesado: generar PDF, agregar datos, etc.
    return { url: 'https://.../reporte.pdf' };
  }
}
```

## Como se usa en un proyecto real

1. Se identifica que trabajo debe moverse a una cola: cualquier cosa lenta, con reintentos necesarios, o que no debe bloquear la respuesta HTTP inmediata.
2. Se configuran reintentos con backoff exponencial para trabajos que pueden fallar temporalmente (llamadas a servicios externos).
3. Los workers pueden desplegarse como un proceso separado del servidor HTTP, escalando cada uno de forma independiente.
4. Se monitorean las colas (jobs fallidos, atascados) con herramientas como Bull Board.
5. Se define una politica de jobs fallidos: reintentar automaticamente, alertar, o mover a una cola de "dead letter" para revision manual.

## Criterio junior

Un junior debe poder agregar un job simple a una cola y crear un processor basico que lo consuma.

## Criterio mid

Un mid debe configurar reintentos y backoff apropiados segun el tipo de fallo esperado, y monitorear el estado de las colas en desarrollo.

## Criterio senior

Un senior debe decidir la arquitectura de workers (mismo proceso vs proceso separado, cuantas colas, prioridades), definir la politica de jobs fallidos del sistema, y dimensionar Redis para el volumen de jobs esperado.

## Errores comunes

- Usar procesamiento sincrono para trabajo que claramente deberia ser background, bloqueando el Event Loop (ver capitulo de fundamentos).
- No configurar reintentos, perdiendo trabajo ante fallos transitorios de servicios externos.
- No monitorear jobs fallidos, dejando trabajo sin completar sin que nadie se entere.
- Correr los workers en el mismo proceso que atiende requests HTTP de alto trafico, compitiendo por recursos.

## Practica

- [ ] Configura una cola BullMQ simple con Redis local.
- [ ] Crea un productor que agregue jobs y un processor que los consuma.
- [ ] Configura reintentos con backoff para un job que simula fallos intermitentes.
- [ ] Investiga como monitorear colas con Bull Board.
- [ ] Anota que trabajo de tu proyecto deberia moverse a una cola.

## Preguntas de repaso

- Que ventaja tiene BullMQ sobre `EventEmitter` para trabajo que debe garantizarse?
- Por que conviene correr workers en un proceso separado del servidor HTTP?
- Que politica aplicarias ante un job que falla repetidamente?
- Como decidirias que trabajo de tu API deberia moverse a una cola?
