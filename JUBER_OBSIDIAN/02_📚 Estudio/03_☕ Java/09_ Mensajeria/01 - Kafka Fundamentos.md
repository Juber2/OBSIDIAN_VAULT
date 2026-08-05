---
tags:
  - Java
  - Kafka
  - Mensajería
Creado: 2026-05-21
Relacionado:
  - Mensajería
Palabras clave:
  - Kafka, topics, particiones, consumer groups, offset, brokers, ZooKeeper, KRaft
---

## 🧠 Arquitectura Kafka

```
Producers → [Kafka Cluster] → Consumers
                  ↓
            [Topics/Particiones]
            [Brokers (servidores)]
            [ZooKeeper/KRaft (coordinación)]
```

## Conceptos clave

| Concepto | Descripción |
|----------|-------------|
| **Topic** | Canal lógico de mensajes (como una tabla) |
| **Partición** | División de un topic — paralelismo |
| **Broker** | Servidor Kafka (un cluster tiene varios) |
| **Producer** | Publica mensajes en topics |
| **Consumer** | Lee mensajes de topics |
| **Consumer Group** | Grupo de consumers que reparten particiones |
| **Offset** | Posición del consumer en una partición |
| **Replication Factor** | Cuántas copias del topic en diferentes brokers |
| **Leader** | La partición que acepta escrituras |
| **ISR** | In-Sync Replicas — brokers que están al día |

## Topics y Particiones

```
Topic "orders" con 3 particiones:

Partición 0: [msg1] [msg4] [msg7] ...
Partición 1: [msg2] [msg5] [msg8] ...
Partición 2: [msg3] [msg6] [msg9] ...

Consumer Group "payment-service" (3 consumers):
  Consumer A → Partición 0
  Consumer B → Partición 1
  Consumer C → Partición 2
  → Paralelismo máximo
```

```
Consumer Group "analytics" (2 consumers):
  Consumer A → Partición 0 + Partición 1
  Consumer B → Partición 2
  → Un consumer tiene 2 particiones
```

> [!important] Máximo paralelismo = número de particiones
> Si tienes 3 particiones y 5 consumers en el mismo grupo, 2 consumers estarán ociosos. Diseña el número de particiones pensando en el escalado futuro.

## Producer — cómo se asigna la partición

```
1. Si el mensaje tiene KEY → partición = hash(key) % numPartitions
   → Garantiza orden para mensajes con la misma key
   → Ejemplo: key = "userId:42" siempre va a la misma partición

2. Sin KEY → round-robin entre particiones
   → Distribución uniforme pero sin orden garantizado

3. Partición explícita → producer puede especificar directamente
```

## Garantías de entrega

| Configuración | Semántica | Rendimiento |
|---------------|-----------|-------------|
| `acks=0` | Fire and forget (puede perder) | Máximo |
| `acks=1` | At-most-once (el leader confirma) | Medio |
| `acks=all` (-1) | At-least-once (todos ISR confirman) | Menor |

```
At-most-once:  puede perder mensajes (acks=0 o 1)
At-least-once: puede duplicar mensajes (acks=all + retry)
Exactly-once:  ni pérdidas ni duplicados (idempotent producer + transactions)
```

## Exactamente-una-vez (Exactly-once)

```java
// Producer idempotente (evita duplicados del producer)
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
props.put(ProducerConfig.ACKS_CONFIG, "all");
props.put(ProducerConfig.RETRIES_CONFIG, Integer.MAX_VALUE);
props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 5);

// Transacciones Kafka (exactly-once end-to-end)
props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "order-service-txn-1");
```

## Retention — cuánto tiempo guarda Kafka los mensajes

```bash
# Por tiempo (por defecto 7 días)
kafka-topics.sh --alter --topic orders \
  --config retention.ms=604800000  # 7 días

# Por tamaño
kafka-topics.sh --alter --topic orders \
  --config retention.bytes=1073741824  # 1GB por partición

# Para siempre (log compaction — guarda solo el último valor por key)
kafka-topics.sh --alter --topic user-profiles \
  --config cleanup.policy=compact
```

## Consumer Offsets

```
Offset = "hasta dónde he leído"

Auto-commit (por defecto, cada 5s):
  → simple pero puede perder mensajes o duplicar

Manual commit:
  → más control, confirmar SOLO cuando el mensaje fue procesado
```

```java
// Manual commit
@KafkaListener(topics = "orders")
public void consume(ConsumerRecord<String, OrderEvent> record,
                     Acknowledgment ack) {
    try {
        processOrder(record.value());
        ack.acknowledge();  // confirmar SOLO si procesó bien
    } catch (Exception e) {
        // no hacer ack → Kafka reintentará
        log.error("Error procesando mensaje, no se hace ack", e);
    }
}
```

## Comandos CLI útiles

```bash
# Crear topic
kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic orders \
  --partitions 3 \
  --replication-factor 1

# Listar topics
kafka-topics.sh --list --bootstrap-server localhost:9092

# Ver detalle del topic
kafka-topics.sh --describe --topic orders --bootstrap-server localhost:9092

# Producir mensajes manualmente
kafka-console-producer.sh --topic orders --bootstrap-server localhost:9092

# Consumir mensajes
kafka-console-consumer.sh \
  --topic orders \
  --from-beginning \
  --bootstrap-server localhost:9092

# Ver offsets del consumer group
kafka-consumer-groups.sh \
  --describe \
  --group payment-service \
  --bootstrap-server localhost:9092
```

## KRaft — Kafka sin ZooKeeper (Kafka 3.3+)

```properties
# Kafka moderno no necesita ZooKeeper
# KRaft es el nuevo modo de consenso integrado en Kafka
# Simplifica el deployment significativamente
```

```yaml
# docker-compose con KRaft (sin ZooKeeper)
kafka:
  image: confluentinc/cp-kafka:7.6.0
  environment:
    KAFKA_NODE_ID: 1
    KAFKA_PROCESS_ROLES: broker,controller
    KAFKA_LISTENERS: PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
    KAFKA_CONTROLLER_QUORUM_VOTERS: 1@localhost:9093
    CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk  # uuid base64
```

## 📌 Notas Rápidas
