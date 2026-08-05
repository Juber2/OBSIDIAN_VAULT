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

# 05 - Microservices Transport TCP Redis Kafka

## Que vas a aprender

En este capitulo vas a conocer el modulo `@nestjs/microservices`: como convertir una aplicacion Nest en un microservicio que se comunica por **TCP, Redis o Kafka**, y el patron **request-response** vs **event-based** entre servicios.

## Explicacion

NestJS permite que la misma aplicacion actue como microservicio, escuchando mensajes en vez de (o ademas de) requests HTTP. El transporte (`TCP`, `Redis`, `Kafka`, `RabbitMQ`, `gRPC`) es intercambiable en la configuracion sin cambiar la logica de los handlers, que se declaran con `@MessagePattern()` (request-response) o `@EventPattern()` (fire-and-forget).

TCP es el mas simple para empezar (bueno para desarrollo o sistemas chicos). Redis agrega pub/sub sencillo. Kafka es la opcion para volumenes altos, particionamiento y garantias de orden/replay de eventos, a costa de mayor complejidad operativa.

## Modelo mental

Pensa `@MessagePattern()` como un endpoint HTTP que en vez de URL usa un nombre de mensaje, y espera una respuesta. Pensa `@EventPattern()` como un anuncio que se dispara sin esperar respuesta, igual que los eventos de dominio del tema anterior, pero cruzando la frontera entre servicios distintos, no solo entre modulos del mismo proceso.

## Ejemplo base

```typescript
// microservicio de ordenes
@Controller()
export class OrdersController {
  @MessagePattern('orders.create')
  create(@Payload() dto: CreateOrderDto) {
    return this.ordersService.create(dto); // responde al llamador
  }

  @EventPattern('payment.completed')
  handlePaymentCompleted(@Payload() data: { orderId: string }) {
    return this.ordersService.markAsPaid(data.orderId); // no responde, solo reacciona
  }
}

// cliente que llama al microservicio
const result = await this.client.send('orders.create', dto).toPromise();
this.client.emit('payment.completed', { orderId });
```

## Como se usa en un proyecto real

1. Se elige el transporte segun el volumen y las garantias necesarias: TCP/Redis para empezar simple, Kafka/RabbitMQ cuando el volumen y la necesidad de garantias de entrega crecen.
2. Se usa `@MessagePattern()` cuando el llamador necesita una respuesta sincrona logica (aunque sea sobre un transporte asincrono), y `@EventPattern()` para notificaciones fire-and-forget.
3. Se maneja explicitamente el timeout y los errores de comunicacion entre servicios, ya que la red puede fallar de formas que no existen en llamadas locales.
4. Se versiona el contrato de mensajes entre servicios igual que se versionaria un contrato HTTP.
5. Se documenta el mapa de comunicacion entre microservicios (quien llama a quien, con que mensajes) para que el sistema distribuido siga siendo comprensible.

## Criterio junior

Un junior debe poder configurar un microservicio Nest simple sobre TCP y definir un handler basico con `@MessagePattern()`.

## Criterio mid

Un mid debe elegir el transporte correcto segun el caso de uso, manejar timeouts y errores de comunicacion entre servicios, y diferenciar cuando usar `@MessagePattern()` vs `@EventPattern()`.

## Criterio senior

Un senior debe diseñar la topologia de comunicacion entre microservicios de todo el sistema, elegir Kafka/RabbitMQ cuando la escala lo justifica, y definir estrategias de resiliencia (circuit breaker, retries, dead letter queues) para comunicacion inter-servicio.

## Errores comunes

- Usar `@MessagePattern()` (request-response) para algo que en realidad es un evento fire-and-forget, acoplando innecesariamente al llamador a esperar respuesta.
- No manejar timeouts en llamadas entre microservicios, dejando que un servicio caido cuelgue al que lo llama indefinidamente.
- Elegir Kafka desde el dia uno para un sistema pequeño, sumando complejidad operativa sin necesidad real todavia.
- No versionar los contratos de mensajes entre servicios, rompiendo consumidores cuando el productor cambia el payload.

## Practica

- [ ] Configura un microservicio Nest simple sobre transporte TCP.
- [ ] Implementa un handler con `@MessagePattern()` y otro con `@EventPattern()`.
- [ ] Simula un timeout en la comunicacion entre servicios y maneja el error correctamente.
- [ ] Compara conceptualmente TCP, Redis y Kafka como transporte para tu caso de uso.
- [ ] Documenta el mapa de comunicacion de un sistema de microservicios hipotetico.

## Preguntas de repaso

- Que diferencia hay entre `@MessagePattern()` y `@EventPattern()`?
- Cuando elegirias Kafka sobre TCP o Redis como transporte?
- Que riesgo tiene no manejar timeouts en comunicacion entre microservicios?
- Por que los contratos de mensajes entre servicios necesitan versionado igual que una API HTTP?
