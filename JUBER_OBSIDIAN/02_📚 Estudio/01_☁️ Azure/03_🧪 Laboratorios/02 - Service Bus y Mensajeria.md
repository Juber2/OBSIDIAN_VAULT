---
tags:
  - Azure
  - Mensajeria
  - ServiceBus
Creado: 2026-08-30
Relacionado:
  - Azure
  - Laboratorios
---

# Service Bus y Mensajería

> Pregunta literal de la vacante de Entelgy: **"¿Cómo procesas mensajes de Service Bus sin duplicar efectos?"**. El radar marca mensajería en 4 de 14 vacantes, casi siempre con la misma pregunta de fondo: idempotencia.

## Queue vs Topic — la decisión de diseño

```
QUEUE (competing consumers)
  Productor ──► [ Queue ] ──► un solo consumidor recibe cada mensaje
                              (varias instancias compiten, no duplican)

TOPIC (publish/subscribe)
  Productor ──► [ Topic ] ──┬──► Subscription A ──► servicio de facturación
                            ├──► Subscription B ──► servicio de notificaciones
                            └──► Subscription C ──► servicio de analítica
                              (cada suscripción recibe SU copia)
```

Regla práctica: **queue** cuando una sola parte debe procesar el trabajo (procesar un pago). **Topic** cuando un hecho de negocio interesa a varios (una orden se creó). Elegir queue y luego necesitar un segundo consumidor obliga a rediseñar; elegir topic "por las dudas" agrega complejidad que quizá nunca uses.

```bash
az servicebus namespace create --resource-group mi-rg --name mi-sb-ns --sku Standard
az servicebus queue create --resource-group mi-rg --namespace-name mi-sb-ns --name ordenes \
  --enable-duplicate-detection true \
  --duplicate-detection-history-time-window PT10M \
  --max-delivery-count 5
```

> `Standard` soporta topics; `Basic` solo queues. Sessions y transacciones requieren Standard o Premium.

## Peek-Lock: por qué el mensaje no se borra al recibirlo

Este es el mecanismo que hace posible la entrega confiable, y una pregunta frecuente:

```
1. Consumidor recibe el mensaje  → queda BLOQUEADO (lock), invisible para otros
2. Consumidor procesa
3a. Complete()  → el mensaje se elimina definitivamente
3b. Abandon()   → se libera el lock, otro consumidor lo toma inmediatamente
3c. DeadLetter()→ va a la DLQ, no se reintenta
3d. (nada)      → vence el lock (LockDuration) y vuelve a la cola automáticamente
```

El modo alternativo, `ReceiveAndDelete`, borra el mensaje al entregarlo: más rápido, pero **si tu proceso crashea, el mensaje se perdió**. Peek-Lock es el default correcto para cualquier operación con efectos reales.

> Si el procesamiento dura más que `LockDuration` (default 30s, máx 5 min), hay que **renovar el lock** o el mensaje se reentrega mientras todavía lo estás procesando — causa clásica de duplicados "inexplicables".

## Consumir con .NET

```csharp
var client = new ServiceBusClient(fqdn, new DefaultAzureCredential());
var processor = client.CreateProcessor("ordenes", new ServiceBusProcessorOptions
{
    MaxConcurrentCalls = 5,
    AutoCompleteMessages = false   // control explícito: nunca completar antes de tiempo
});

processor.ProcessMessageAsync += async args =>
{
    var mensaje = args.Message;
    var messageId = mensaje.MessageId;

    if (await _procesados.YaProcesadoAsync(messageId))
    {
        await args.CompleteMessageAsync(mensaje);   // duplicado: descartar sin re-ejecutar
        return;
    }

    await _servicio.ProcesarOrdenAsync(mensaje.Body.ToObjectFromJson<OrdenDto>());
    await _procesados.RegistrarAsync(messageId);
    await args.CompleteMessageAsync(mensaje);
};

processor.ProcessErrorAsync += args =>
{
    _logger.LogError(args.Exception, "Error procesando mensaje");
    return Task.CompletedTask;
};

await processor.StartProcessingAsync();
```

`AutoCompleteMessages = false` es deliberado: con `true`, el SDK completa el mensaje aunque tu lógica haya fallado silenciosamente.

## Consumir con Java/Spring

```java
ServiceBusProcessorClient processor = new ServiceBusClientBuilder()
    .fullyQualifiedNamespace(fqdn)
    .credential(new DefaultAzureCredentialBuilder().build())
    .processor()
    .queueName("ordenes")
    .disableAutoComplete()
    .processMessage(context -> {
        ServiceBusReceivedMessage msg = context.getMessage();
        if (procesados.yaProcesado(msg.getMessageId())) {
            context.complete();
            return;
        }
        servicio.procesarOrden(msg.getBody().toString());
        procesados.registrar(msg.getMessageId());
        context.complete();
    })
    .processError(ctx -> log.error("Error", ctx.getException()))
    .buildProcessorClient();

processor.start();
```

## Idempotencia — la respuesta completa a la pregunta de entrevista

Service Bus garantiza **at-least-once**, no exactly-once. El duplicado no es un bug: es el contrato. Hay tres capas de defensa, y la respuesta fuerte las menciona todas:

| Capa | Qué hace | Límite |
|---|---|---|
| **Duplicate detection** de Service Bus | Descarta mensajes con el mismo `MessageId` dentro de una ventana de tiempo | Solo cubre duplicados del **productor**, y solo dentro de la ventana (máx 7 días) |
| **Registro de procesados** en el consumidor | Guardás el `MessageId` en base y verificás antes de actuar | Es lo que cubre las reentregas por lock vencido o crash |
| **Operación naturalmente idempotente** | Asignar un estado final en vez de sumar/restar sobre el valor actual | La más robusta: no necesita estado extra |

Sobre la tercera capa, la diferencia concreta:

```sql
-- ❌ No idempotente: ejecutarlo dos veces descuenta doble
UPDATE cuenta SET saldo = saldo - 100 WHERE id = 7;

-- ✅ Idempotente: ejecutarlo dos veces deja el mismo resultado
UPDATE orden SET estado = 'PAGADO' WHERE id = 42;
```

> La mejor respuesta en entrevista: *"Diseño la operación para que sea idempotente por naturaleza cuando puedo; cuando no, registro el MessageId procesado en la misma transacción que el efecto de negocio."* Ese "en la misma transacción" es lo que diferencia una respuesta memorizada de una vivida — si registrás el ID en otra transacción, existe la ventana donde procesaste pero no registraste.

## Dead Letter Queue

Un mensaje va a la DLQ cuando supera `MaxDeliveryCount`, expira su TTL, o lo mandás explícitamente. **La DLQ no es un log de errores: es una cola que alguien tiene que mirar.** Un sistema con DLQ que nadie monitorea es un sistema que pierde mensajes en silencio.

```csharp
await args.DeadLetterMessageAsync(mensaje,
    deadLetterReason: "ValidacionFallida",
    deadLetterErrorDescription: "El cliente no existe");
```

Enviar a DLQ *inmediatamente* cuando el error no es transitorio (un cliente inexistente no va a existir en el reintento 5) es mejor que gastar 5 reintentos. Reservá los reintentos para fallos transitorios: red, timeout, deadlock.

```bash
# Cuántos mensajes hay muertos en la DLQ
az servicebus queue show --resource-group mi-rg --namespace-name mi-sb-ns --name ordenes \
  --query "countDetails.deadLetterMessageCount"
```

## Sessions: cuando el orden importa

Por default Service Bus **no garantiza orden** si hay varios consumidores. Si necesitás que todos los eventos de la orden #123 se procesen en secuencia, usás sessions con `SessionId = "orden-123"`: Service Bus entrega esa sesión a un solo consumidor a la vez, en orden.

Costo: perdés paralelismo dentro de la sesión. Solo activalo si el orden es un requisito real, no por precaución.

## Cómo se conecta con Outbox

Service Bus resuelve la entrega, pero no el problema anterior: **¿cómo garantizás que el mensaje se publique si y solo si la transacción de negocio se confirmó?** Guardar en base y publicar son dos sistemas distintos; si el proceso muere entre ambos, quedás inconsistente.

Eso lo resuelve el **patrón Outbox** (documentado en las notas de Mensajería de Java): escribís el evento en una tabla dentro de la misma transacción de negocio, y un proceso aparte lo publica. Service Bus + Outbox + idempotencia en el consumidor es la tríada completa que esperan escuchar en una entrevista senior.

## Equivalencias (para vacantes que piden otro broker)

| Service Bus | Kafka | RabbitMQ |
|---|---|---|
| Queue | Topic con un consumer group | Queue |
| Topic + Subscriptions | Topic con varios consumer groups | Exchange (fanout/topic) + queues |
| Peek-Lock / Complete | Commit de offset | ack / nack |
| Dead Letter Queue | Dead Letter Topic | Dead Letter Exchange (DLX) |
| Sessions | Partición por key | Consistent hash / single active consumer |

Los conceptos son transferibles; en entrevista conviene nombrar el equivalente del broker que ellos usan.

## Errores comunes

- Asumir exactly-once. No existe en la práctica: se construye con at-least-once + idempotencia.
- `AutoComplete = true` con lógica que puede fallar — se completa el mensaje aunque el negocio no se haya ejecutado.
- Procesar más tiempo que el `LockDuration` sin renovar el lock: el mensaje se reentrega mientras seguís procesándolo.
- Registrar el `MessageId` procesado en una transacción distinta a la del efecto de negocio.
- Tener DLQ sin alerta ni proceso de revisión.
- Activar sessions "por si acaso", sacrificando throughput sin necesitarlo.

## Checklist

- [ ] Sé cuándo usar queue y cuándo topic, y qué cuesta cambiar después.
- [ ] Puedo explicar peek-lock y qué pasa si vence el lock.
- [ ] Puedo dar las tres capas de idempotencia sin leer.
- [ ] Sé por qué el registro de procesados debe ir en la misma transacción que el efecto.
- [ ] Sé cuándo mandar a DLQ de inmediato en vez de reintentar.
- [ ] Puedo mapear Service Bus a Kafka y RabbitMQ concepto por concepto.
- [ ] Puedo explicar qué problema resuelve Outbox que Service Bus no resuelve solo.
