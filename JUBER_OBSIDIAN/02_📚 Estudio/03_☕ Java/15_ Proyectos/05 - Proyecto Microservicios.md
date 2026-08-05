---
tags:
  - Java
  - Microservicios
  - Proyecto
Creado: 2026-05-21
Relacionado:
  - Proyectos Java
---

# Proyecto Microservicios

## Objetivo

Construir un sistema pequeno con servicios separados y comunicacion sincrona/asincrona.

## Servicios sugeridos

- `user-service`
- `catalog-service`
- `order-service`
- `payment-service`
- `notification-service`

## Requisitos

- API Gateway.
- Service discovery o configuracion estatica local.
- Kafka o RabbitMQ para eventos.
- Circuit breaker.
- Distributed tracing.
- Docker Compose.

## Checklist

- [ ] Cada servicio tiene su propia base de datos o esquema.
- [ ] Hay eventos de dominio.
- [ ] Hay resiliencia ante fallos.
- [ ] Puedo levantar todo localmente.
