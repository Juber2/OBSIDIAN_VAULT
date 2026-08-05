---
tags:
  - dotnet
  - Senior
  - devops
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Cloud DevOps Mensajeria dotNET
  - NET
---

# Mensajeria RabbitMQ Azure Service Bus Kafka

## Cuándo usar mensajes

Cuando necesitas desacoplar, procesar lento en background, tolerar picos o integrar sistemas.

## Comparativa

RabbitMQ: colas y routing. Azure Service Bus: colas/topics administrados. Kafka: log distribuido/event streaming.

## Reglas

Mensajes deben ser idempotentes, versionables y observables. Maneja retries, dead-letter y poison messages.

## Outbox

Evita perder eventos entre DB y broker guardando evento en la misma transaccion y publicando después.

## Errores comunes

- Publicar evento antes de guardar DB
- Consumidores no idempotentes
- No tener DLQ

## Practica

- [ ] Diseñar evento UserCreated y outbox

## Relacionado

- [[Cloud DevOps Mensajeria dotNET]]
- [[NET - Roadmap]]
