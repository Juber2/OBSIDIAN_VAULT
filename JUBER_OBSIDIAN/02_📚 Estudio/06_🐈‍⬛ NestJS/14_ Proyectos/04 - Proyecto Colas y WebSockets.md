---
tags:
  - NestJS
  - TypeScript
  - Nest-Proyectos
Creado: 2026-07-27
Relacionado:
  - Proyectos
  - NESTJS
---

# 04 - Proyecto Colas y WebSockets

## Que vas a construir

Un modulo de notificaciones en tiempo real sobre el proyecto SaaS anterior: procesamiento en background con BullMQ y entrega en tiempo real al cliente conectado via WebSockets, con una tarea programada de limpieza.

## Alcance funcional

- Un evento de dominio (por ejemplo, "orden creada") que dispara un job en background (BullMQ) para procesar algo pesado (generar un resumen, enviar un email simulado).
- Notificacion en tiempo real al usuario conectado via WebSocket Gateway cuando ese job termina.
- Autenticacion de la conexion WebSocket (no cualquiera puede conectarse y escuchar).
- Una tarea programada (`@Cron`) que limpia datos expirados, con logging de cada ejecucion.
- Reintentos configurados en el job de BullMQ para simular resiliencia ante fallos transitorios.

## Checklist tecnica

- [ ] El evento de dominio esta desacoplado del listener que encola el job (no se llama al servicio de colas directo desde el flujo principal).
- [ ] El job de BullMQ tiene reintentos con backoff configurados, probados forzando un fallo simulado.
- [ ] La conexion WebSocket rechaza clientes sin token valido en `handleConnection`.
- [ ] Se usan rooms para que un usuario solo reciba notificaciones que le corresponden a el (o a su tenant).
- [ ] La tarea programada loggea cada ejecucion (exito o fallo) de forma auditable.
- [ ] Se documenta (aunque no se implemente) como se evitaria ejecucion duplicada de la tarea programada si la app corriera en multiples instancias.

## Criterio de exito

El proyecto se considera completo cuando, desde un cliente de prueba conectado por WebSocket, se puede disparar la accion que crea el job, ver que el job se procesa en background (con logs), y recibir la notificacion en tiempo real sin que el cliente haga polling.

## Errores comunes a evitar

- Ejecutar el trabajo pesado de forma sincrona en el mismo request en vez de encolarlo, bloqueando el Event Loop.
- No autenticar la conexion WebSocket, permitiendo que cualquiera se conecte y reciba notificaciones ajenas.
- No configurar reintentos en el job, perdiendo trabajo ante un fallo transitorio simulado.
- Enviar la notificacion a todos los clientes conectados en vez de solo al room correspondiente.

## Extension opcional

Agregar un dashboard simple (aunque sea un cliente de prueba en HTML) que muestre notificaciones en tiempo real, para verificar el flujo completo de punta a punta de forma visual.
