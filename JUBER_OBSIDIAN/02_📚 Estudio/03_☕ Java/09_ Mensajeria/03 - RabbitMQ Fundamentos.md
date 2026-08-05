---
tags:
  - Java
  - RabbitMQ
  - Mensajería
Creado: 2026-05-21
Relacionado:
  - Mensajería
Palabras clave:
  - RabbitMQ, AMQP, exchanges, queues, bindings, routing key, DLX
---

## 🧠 Arquitectura RabbitMQ

```
Producer → Exchange → (Binding/Routing Key) → Queue → Consumer
```

A diferencia de Kafka, el mensaje se enruta a través de un **Exchange** antes de llegar a la cola.

## Tipos de Exchange

### Direct Exchange
```
Producer → Exchange(direct) → routing_key="orders.create" → Queue "orders-create"
                            → routing_key="orders.cancel" → Queue "orders-cancel"
```

### Fanout Exchange
```
Producer → Exchange(fanout) → Queue A (todos reciben el mismo mensaje)
                            → Queue B
                            → Queue C
```

### Topic Exchange
```
Producer → Exchange(topic) → routing_key="orders.#"   → Queue "all-orders"
                           → routing_key="orders.*.es" → Queue "spanish-orders"
                           → routing_key="*.error"     → Queue "all-errors"

# Wildcards:
# * (asterisco) → exactamente una palabra
# # (hash) → cero o más palabras
```

### Headers Exchange
```
Enruta por headers del mensaje en lugar del routing key
Menos común, más flexible
```

## Cuándo usar cada exchange

| Exchange | Usar cuando |
|----------|-------------|
| **Direct** | Routing preciso por clave (task queues) |
| **Fanout** | Notificar a todos los suscriptores (eventos de dominio) |
| **Topic** | Routing por patrón (logs por nivel/servicio) |
| **Default** | El exchange "" — directo al queue por nombre |

## Conceptos adicionales

### Durable, Persistent, Auto-delete

```bash
# Queue durable → sobrevive al reinicio del broker
# Message persistent → sobrevive al reinicio
# Auto-delete → se elimina cuando el último consumer desconecta
```

### Dead Letter Exchange (DLX)

```
Si un mensaje falla (rejected, expired, o queue llena):
  → va al Dead Letter Exchange
  → desde ahí a una Dead Letter Queue
  → para análisis, reintento manual, o descarte
```

### Message TTL y Queue TTL

```bash
# TTL del mensaje (expirar si no se consume)
x-message-ttl: 60000  # 60 segundos

# TTL de la queue (eliminar si no tiene consumers por X tiempo)
x-expires: 3600000   # 1 hora
```

### Prefetch / QoS

```bash
# Cuántos mensajes puede tener un consumer sin confirmar
# Sin prefetch: RabbitMQ envía todos al consumer → saturación
# Con prefetch=1: solo 1 mensaje a la vez → fair dispatch
channel.basicQos(1)
```

## Comparación de mensajes procesados

```
Kafka:
  - El consumer PULL (pide mensajes)
  - Offset persiste en Kafka
  - Puede releer mensajes históricos

RabbitMQ:
  - El broker PUSH (envía al consumer)
  - Mensaje se elimina al hacer ACK
  - No se puede releer (a menos que uses DLQ)
```

## Reliable Publishing

```
Producer Confirms: el broker confirma que recibió el mensaje
  → channel.confirmSelect()
  → channel.addConfirmListener()

Persistent messages: el mensaje sobrevive al reinicio del broker
  → MessageProperties.PERSISTENT_TEXT_PLAIN
```

## Patrones de uso

### Work Queue (Task Queue)
```
Producer → Queue → Consumer A
                 → Consumer B  (round-robin entre consumers)

Ideal para: distribuir trabajo entre workers
```

### Publish/Subscribe
```
Producer → Fanout Exchange → Queue A → Consumer A (servicio de email)
                           → Queue B → Consumer B (servicio de SMS)
                           → Queue C → Consumer C (servicio de push)

Ideal para: notificaciones, eventos
```

### RPC (Request/Reply)
```
Client → Request Queue → Server
Server → Reply Queue → Client (usando correlationId)

Ideal para: llamadas request/response async
```

## 📌 Notas Rápidas
