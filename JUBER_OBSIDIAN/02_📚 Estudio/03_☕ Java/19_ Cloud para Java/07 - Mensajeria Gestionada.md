---
tags:
  - Java
  - Cloud
  - Mensajeria
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - Mensajeria
Palabras clave:
  - Service Bus, SQS, SNS, Event Hubs, Kinesis, MSK, visibility timeout, DLQ, FIFO
---

## 🧠 Concepto

Tres familias de servicios que se confunden todo el tiempo porque "todos mandan mensajes". Resuelven problemas distintos:

| Familia | Modelo | Un mensaje lo recibe | Se guarda tras leerlo |
|---|---|---|---|
| **Cola** | punto a punto | **un** consumidor | ❌ se borra |
| **Pub/sub** | uno a muchos | **cada** suscriptor | ❌ se entrega y se descarta |
| **Streaming** | log ordenado | cualquiera que lea la partición | ✅ por retención |

```
Cola       → repartir trabajo         "que alguien procese este pago"
Pub/sub    → notificar                "ocurrió una transferencia, avisen a quien le importe"
Streaming  → historia reproducible    "todos los movimientos, releíbles desde el principio"
```

> [!tip] La diferencia práctica es si podés releer
> Si un consumidor nuevo tiene que procesar eventos **anteriores** a su existencia —una proyección nueva, un reproceso, un análisis histórico— necesitás streaming. Con una cola, lo que se consumió se fue.

## 🗄️ El catálogo

| Necesito | Azure | AWS |
|---|---|---|
| Cola punto a punto | **Service Bus Queue** | **SQS** |
| Cola simple y barata | Storage Queue | SQS |
| Pub/sub | Service Bus Topic | **SNS** |
| Enrutamiento de eventos | Event Grid | **EventBridge** |
| Streaming de eventos | **Event Hubs** | Kinesis Data Streams |
| Kafka gestionado | Event Hubs (endpoint Kafka) | **MSK** |

> [!tip] Event Hubs habla el protocolo de Kafka
> Si ya tenés código con `spring-kafka`, apunta a Event Hubs cambiando la configuración de conexión, sin reescribir consumidores ni productores. Es una vía de migración cómoda desde un Kafka propio.

## 📬 Colas: el modelo de bloqueo

La diferencia conceptual más importante con Kafka: en una cola **el mensaje se bloquea, se procesa y se borra**. No hay offsets.

```
1 · El consumidor recibe el mensaje → queda invisible para los demás
2 · Tiene N segundos para procesarlo
3 · Confirma → el mensaje se borra
4 · Si no confirma a tiempo → vuelve a la cola y otro lo toma
```

| Concepto | Azure Service Bus | AWS SQS |
|---|---|---|
| Ventana de proceso | *lock duration* | *visibility timeout* |
| Confirmar | `complete()` | `DeleteMessage` |
| Devolver | `abandon()` | cambiar visibilidad a 0 |
| Extender | `renewLock()` | `ChangeMessageVisibility` |
| Rechazar a DLQ | `deadLetter()` | tras N intentos |

```java
@JmsListener(destination = "transferencias")
public void alRecibir(TransferenciaMensaje mensaje, Message jmsMessage) {
    procesar(mensaje);
    // con AUTO_ACKNOWLEDGE, el retorno sin excepción confirma
}
```

```java
// AWS SQS con Spring Cloud AWS
@SqsListener("transferencias")
public void alRecibir(TransferenciaMensaje mensaje, Acknowledgement ack) {
    procesar(mensaje);
    ack.acknowledge();
}
```

> [!danger] Si el procesamiento tarda más que la ventana, el mensaje se duplica
> El servicio asume que moriste y se lo entrega a otro consumidor — **mientras vos seguís procesándolo**. Dos consumidores trabajando sobre el mismo mensaje al mismo tiempo.
>
> Dos defensas, y hacen falta las dos:
> 1. Ventana mayor al p99 del tiempo de procesamiento, o renovarla mientras se trabaja.
> 2. **Idempotencia del consumidor** — ver [[03 - Idempotencia y Deduplicacion]].

```java
// Renovar el bloqueo en trabajos largos
var renovador = Executors.newSingleThreadScheduledExecutor();
var tarea = renovador.scheduleAtFixedRate(
    () -> receptor.renewMessageLock(mensaje), 20, 20, TimeUnit.SECONDS);
try {
    procesarLargo(mensaje);
} finally {
    tarea.cancel(true);
}
```

### Orden

| | Cómo se ordena |
|---|---|
| Service Bus | **sesiones**: los mensajes con el mismo `SessionId` se entregan en orden a un solo consumidor |
| SQS FIFO | **grupos**: orden dentro de cada `MessageGroupId` |
| Kafka / Event Hubs | orden dentro de la partición, por clave |

```java
// Service Bus: todos los mensajes de una cuenta, en orden
var mensaje = new ServiceBusMessage(payload);
mensaje.setSessionId(cuentaId.toString());
```

> [!warning] El orden cuesta rendimiento en las tres opciones
> Ordenar implica que un solo consumidor procese ese grupo por vez. SQS FIFO tiene un techo de tasa bastante menor que el estándar; las sesiones de Service Bus serializan por sesión.
> Es el mismo compromiso de [[04 - Garantias de Entrega]]: **orden y paralelismo se pelean**. La granularidad correcta suele ser "orden por agregado".

## 📡 Pub/sub y abanico

```
                    ┌──► SQS: notificaciones
SNS "movimientos" ──┼──► SQS: antifraude
                    ├──► SQS: proyecciones
                    └──► Lambda: auditoría
```

> [!tip] El patrón SNS → SQS es el estándar de AWS
> SNS solo entrega: si el destino no responde, el mensaje se pierde. Poniendo una cola SQS por consumidor, cada uno tiene su propio buffer, sus reintentos y su DLQ, y puede estar caído sin afectar a los demás.
> El equivalente en Azure es un Topic con una suscripción por consumidor — que ya funciona así de forma nativa.

```java
// Service Bus: filtrar del lado del servidor
// Cada suscripción recibe solo lo que le corresponde
subscription.createRule("solo-altas",
    new CreateRuleOptions(new SqlRuleFilter("tipo = 'TRANSFERENCIA' AND monto > 10000")));
```

> [!tip] Filtrar en el servidor ahorra dinero y trabajo
> Un consumidor que recibe todo y descarta el 95% paga el transporte de todo. Con reglas de filtrado, el servicio entrega solo lo relevante.

## 🌊 Streaming

```yaml
# Event Hubs con el protocolo de Kafka: el mismo código de spring-kafka
spring:
  kafka:
    bootstrap-servers: mieventhub.servicebus.windows.net:9093
    properties:
      security.protocol: SASL_SSL
      sasl.mechanism: OAUTHBEARER          # con identidad gestionada
```

| | Event Hubs | Kinesis | MSK |
|---|---|---|---|
| Unidad de paralelismo | partición | *shard* | partición |
| Retención | hasta 7 días (más con Premium) | hasta 365 días | configurable |
| Compatible con Kafka | ✅ | ❌ | ✅ es Kafka |
| Escalado | unidades de rendimiento | shards | tamaño del clúster |

> [!warning] El número de particiones limita el paralelismo
> Un grupo de consumidores no puede tener más lectores activos que particiones: los sobrantes quedan ociosos. Y en varios de estos servicios **reducir particiones no se puede** — hay que crear un recurso nuevo y migrar.
> Es una decisión de dimensionamiento que conviene pensar al crear, no después.

## ☠️ Cola de mensajes fallidos

```java
// Service Bus: rechazo explícito con motivo
receptor.deadLetter(mensaje, new DeadLetterOptions()
    .setDeadLetterReason("PAYLOAD_INVALIDO")
    .setDeadLetterErrorDescription(ex.getMessage()));
```

```json
// SQS: política de redirección tras N intentos
{
  "RedrivePolicy": {
    "deadLetterTargetArn": "arn:aws:sqs:us-east-1:123:transferencias-dlq",
    "maxReceiveCount": 3
  }
}
```

| Motivo | Qué hacer |
|---|---|
| Error transitorio | reintentar con backoff |
| Payload inválido | a la DLQ sin reintentar |
| Mensaje envenenado | DLQ + alerta |
| Expiró | DLQ |

> [!danger] Una DLQ sin alerta es un agujero negro
> Los mensajes se acumulan en silencio: la aplicación funciona, las métricas se ven bien, y hay transferencias sin notificar desde hace tres días. Necesitás **métrica de profundidad con alerta** y un procedimiento de reproceso documentado.

```java
@Scheduled(fixedRate = 60_000)
public void vigilarDlq() {
    long profundidad = cliente.contarMensajes("transferencias-dlq");
    metricas.gauge("dlq.profundidad", profundidad);
    if (profundidad > 0) alertas.advertencia("hay mensajes en la DLQ", profundidad);
}
```

## 🔁 Deduplicación

| | Azure Service Bus | AWS SQS FIFO |
|---|---|---|
| Mecanismo | detección de duplicados por `MessageId` | `MessageDeduplicationId` |
| Ventana | configurable (hasta 7 días) | 5 minutos |

> [!warning] La deduplicación del servicio no reemplaza la del consumidor
> La ventana es corta y solo cubre duplicados del **productor**. Un mensaje reentregado porque venció la ventana de bloqueo pasa igual. La idempotencia del consumidor sigue siendo obligatoria — ver [[05 - Transactional Outbox]] y el patrón Inbox.

## 💰 Costo

| Modelo | Azure | AWS |
|---|---|---|
| Por operación | Service Bus Standard | SQS, SNS |
| Por capacidad reservada | Service Bus Premium | MSK, Kinesis por shard |
| Por volumen ingresado | Event Hubs | Kinesis |

> [!tip] El sondeo constante se factura
> En SQS, cada consulta a una cola vacía es una operación facturable. Con *long polling* (`WaitTimeSeconds: 20`) el consumidor espera hasta 20 segundos por un mensaje en vez de preguntar en bucle: menos costo y menos latencia al mismo tiempo.

```yaml
spring:
  cloud:
    aws:
      sqs:
        listener:
          poll-timeout: 20s          # long polling
```

## 🎯 Cómo elegir

```
¿Necesito releer mensajes viejos o que varios consumidores lean el mismo flujo?
├── SÍ → streaming (Event Hubs / Kinesis / MSK)
└── NO → ¿un mensaje lo procesa uno o varios?
         ├── uno    → cola (Service Bus Queue / SQS)
         └── varios → pub/sub (Topic / SNS→SQS)
```

| Caso | Servicio |
|---|---|
| Repartir trabajo entre workers | cola |
| Notificar a 3 sistemas | pub/sub |
| Eventos de dominio reproducibles | streaming |
| Telemetría de alto volumen | streaming |
| Tareas con orden por cliente | cola con sesiones/grupos |
| Reaccionar a eventos de infraestructura | Event Grid / EventBridge |

## 🧪 Tests

```java
@Test
void un_mensaje_no_confirmado_vuelve_a_la_cola() {
    enviar(mensaje);
    recibirSinConfirmar();
    esperar(ventanaDeBloqueo.plusSeconds(2));
    assertTrue(recibir().isPresent());
}

@Test
void tras_N_intentos_fallidos_el_mensaje_va_a_la_DLQ() {
    enviarMensajeQueSiempreFalla();
    esperarQue(() -> profundidadDlq() == 1);
}

@Test
void el_procesamiento_es_idempotente_ante_reentrega() {
    consumidor.procesar(mensaje);
    consumidor.procesar(mensaje);
    verify(manejador, times(1)).ejecutar(any());
}

@Test
void los_mensajes_de_una_cuenta_se_procesan_en_orden() { ... }
```

Con **LocalStack** (AWS) o el emulador de Service Bus se prueba de verdad en Testcontainers.

## ⚠️ Errores Comunes

> [!warning] Los que producen duplicados o pérdidas
> - **Ventana de bloqueo menor** al tiempo real de procesamiento: duplicados garantizados.
> - **Consumidor no idempotente**: la reentrega duplica el efecto.
> - **SNS sin colas intermedias**: si el consumidor está caído, se pierde el mensaje.
> - **Sin DLQ**, o con DLQ sin monitorear.
> - **Confiar solo en la deduplicación del servicio**: ventana corta, cubre poco.
> - **Sondeo corto en SQS**: costo y latencia innecesarios.
> - **Elegir streaming cuando alcanzaba una cola** (o al revés).
> - **Pocas particiones**: limita el paralelismo y a veces no se puede corregir.

## 📌 Notas Rápidas
- La diferencia mental clave entre cola y streaming: en una cola el mensaje **se consume y desaparece**; en un stream **se lee y sigue ahí**. Todo lo demás se deriva de eso.
- El patrón SNS → SQS (abanico con buffer por consumidor) es una de las preguntas frecuentes de entrevista sobre AWS, y la razón —aislar fallos entre consumidores— es lo que importa.
- Toda la teoría de [[04 - Garantias de Entrega]] aplica igual acá: el servicio gestionado cambia la operación, no las garantías.
- Se aplica en [[Fase 5 - Eventos y Outbox]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[06 - Bases de Datos Gestionadas]]
- **Siguiente**: [[08 - Almacenamiento de Objetos]]
- **Se conecta con**: [[Mensajeria]], [[02 - Service Bus y Mensajeria]], [[04 - Garantias de Entrega]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

