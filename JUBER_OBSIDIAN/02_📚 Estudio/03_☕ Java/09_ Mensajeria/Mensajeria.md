---
tags:
  - Java
  - Mensajería
  - Kafka
  - RabbitMQ
Creado: 2026-05-21
Relacionado:
  - Microservicios
  - Spring Boot
Palabras clave:
  - Kafka, RabbitMQ, message broker, pub/sub, cola de mensajes, eventos
---

#Principal

# 📨 Mensajería

## Progreso

| Tema | Estado |
|------|--------|
| [[01 - Kafka Fundamentos]] | ⬜ |
| [[02 - Kafka con Spring Boot]] | ⬜ |
| [[03 - RabbitMQ Fundamentos]] | ⬜ |
| [[04 - RabbitMQ con Spring Boot]] | ⬜ |

## Kafka vs RabbitMQ

| | Kafka | RabbitMQ |
|-|-------|----------|
| Modelo | Log distribuido (pub/sub + streams) | Message broker (queues + exchanges) |
| Retención | Permanente (configurable) | Temporal (hasta consumir) |
| Orden | Por partición | Por cola |
| Throughput | Altísimo (millones/s) | Alto (miles/s) |
| Consumer | Pull (el consumer pide) | Push (broker envía) |
| Replay | ✅ Sí (rewind offset) | ❌ No |
| Caso de uso | Event streaming, auditoría, big data | Task queues, RPC, routing complejo |
| Ideal para | Microservicios con eventos | Jobs, notificaciones, workflows |
