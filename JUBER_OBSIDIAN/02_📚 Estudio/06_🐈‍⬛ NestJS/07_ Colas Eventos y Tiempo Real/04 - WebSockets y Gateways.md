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

# 04 - WebSockets y Gateways

## Que vas a aprender

En este capitulo vas a aprender a implementar comunicacion en tiempo real con NestJS usando **Gateways** (`@WebSocketGateway()`), sobre Socket.IO o WS nativo, para casos como chat, notificaciones en vivo o dashboards que se actualizan solos.

## Explicacion

Un Gateway en Nest es el equivalente de un controller pero para conexiones WebSocket: define handlers para eventos entrantes (`@SubscribeMessage('mensaje')`) y puede emitir eventos hacia uno o varios clientes conectados. Por default usa Socket.IO, que agrega reconexion automatica y fallback a polling; tambien se puede usar el adaptador `ws` nativo para menor overhead si no se necesitan esas features.

Los guards e interceptors tambien aplican a Gateways, pero el manejo de autenticacion es distinto: no hay headers HTTP en cada mensaje, la autenticacion tipicamente se hace una vez en la conexion (via query param o header inicial) y se guarda en el socket.

## Modelo mental

Pensa un Gateway como un controller que en vez de responder una vez por request, mantiene una linea abierta con el cliente: puede recibir varios mensajes y enviar varios mensajes por esa misma conexion, en cualquier momento, sin que el cliente tenga que volver a pedir.

## Ejemplo base

```typescript
@WebSocketGateway({ cors: { origin: '*' } })
export class ChatGateway {
  @WebSocketServer() server: Server;

  @SubscribeMessage('mensaje')
  handleMensaje(@MessageBody() data: { texto: string }, @ConnectedSocket() client: Socket) {
    this.server.to(data['room']).emit('mensaje', { texto: data.texto, de: client.id });
  }

  handleConnection(client: Socket) {
    const token = client.handshake.query.token as string;
    // validar token y asociar usuario al socket
  }
}
```

## Como se usa en un proyecto real

1. La autenticacion se valida en `handleConnection`, rechazando (`client.disconnect()`) conexiones sin credenciales validas.
2. Se usan **rooms** de Socket.IO para agrupar clientes (por ejemplo, todos los usuarios viendo la misma orden) y emitir solo a ese grupo.
3. Para escalar horizontalmente con multiples instancias, se usa un adaptador de Redis para Socket.IO, que sincroniza eventos entre instancias.
4. Se valida cada mensaje entrante igual que un endpoint HTTP (DTOs, `class-validator`), porque el cliente sigue siendo una fuente no confiable.
5. Se decide caso por caso si conviene WebSockets (bidireccional, persistente) o alternativas mas simples como polling o Server-Sent Events, segun la necesidad real de tiempo real.

## Criterio junior

Un junior debe poder crear un Gateway simple que reciba y emita mensajes basicos a los clientes conectados.

## Criterio mid

Un mid debe implementar autenticacion en la conexion, usar rooms para segmentar comunicacion, y validar los datos de los mensajes entrantes igual que en un endpoint HTTP.

## Criterio senior

Un senior debe diseñar la arquitectura de tiempo real del sistema completo (que necesita WebSockets vs polling vs SSE), planificar el escalamiento horizontal con adaptador de Redis, y definir la estrategia de reconexion y perdida de mensajes.

## Errores comunes

- No validar ni autenticar la conexion WebSocket, permitiendo que cualquiera se conecte y escuche o emita mensajes.
- No usar rooms y enviar todos los eventos a todos los clientes conectados, generando trafico innecesario y filtrando informacion entre usuarios que no deberian verla.
- Escalar a multiples instancias sin adaptador de Redis, rompiendo la comunicacion entre clientes conectados a instancias distintas.
- No validar el payload de los mensajes entrantes, tratando al cliente WebSocket como si fuera confiable por default.

## Practica

- [ ] Crea un Gateway simple con un evento de ida y vuelta.
- [ ] Implementa autenticacion en `handleConnection` rechazando conexiones invalidas.
- [ ] Implementa rooms para segmentar la comunicacion entre grupos de clientes.
- [ ] Valida el payload de un mensaje entrante con un DTO y `class-validator`.
- [ ] Investiga como configurar el adaptador de Redis para escalar Socket.IO.

## Preguntas de repaso

- En que se diferencia un Gateway de un controller HTTP tradicional?
- Como se maneja tipicamente la autenticacion en una conexion WebSocket?
- Para que sirven las rooms de Socket.IO?
- Que problema resuelve el adaptador de Redis al escalar WebSockets a multiples instancias?
