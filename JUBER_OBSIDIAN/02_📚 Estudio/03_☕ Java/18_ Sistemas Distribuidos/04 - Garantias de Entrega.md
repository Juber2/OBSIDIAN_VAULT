---
tags:
  - Java
  - Distribuido
  - Mensajeria
Creado: 2026-09-08
Relacionado:
  - Sistemas Distribuidos
  - 03 - Idempotencia y Deduplicacion
Palabras clave:
  - at-least-once, at-most-once, exactly-once, dos generales, acks, offsets, orden, Kafka
---

## 🧠 Concepto

Cuando un sistema envía un mensaje a otro, hay exactamente tres garantías posibles:

| Garantía | Significa | Se pierden | Se duplican |
|---|---|---|---|
| **At-most-once** | como mucho una vez | ✅ sí | ❌ no |
| **At-least-once** | al menos una vez | ❌ no | ✅ sí |
| **Exactly-once** | exactamente una vez | ❌ no | ❌ no |

Y hay un resultado teórico incómodo: **exactly-once no es alcanzable a nivel de transporte**.

## 🎖️ El problema de los dos generales

Dos generales tienen que atacar simultáneamente. Solo pueden comunicarse por mensajeros que pueden ser interceptados.

```
General A → "atacamos al amanecer" → ¿llegó?
   A no sabe si llegó, así que pide confirmación.
General B → "confirmado" → ¿llegó?
   B no sabe si su confirmación llegó, así que A podría no atacar.
   A confirma la confirmación → ¿llegó?
   ...
```

> [!danger] No hay solución
> Está demostrado: sobre un canal no confiable **no existe protocolo** que garantice acuerdo con certeza, sin importar cuántos mensajes se intercambien. Siempre queda un último mensaje del que no se tiene confirmación.
>
> Toda la mensajería distribuida vive dentro de esta limitación. Por eso hay que **elegir** entre perder o duplicar.

## ⚖️ La elección

### At-most-once — enviar y olvidar

```java
kafkaTemplate.send("eventos", evento);      // sin esperar confirmación
```

Rápido y barato. Aceptable para métricas, telemetría o logs donde perder una muestra no importa. **Inaceptable** para cualquier cosa con efecto de negocio.

### At-least-once — el estándar

```java
kafkaTemplate.send("eventos", evento)
    .get(5, TimeUnit.SECONDS);      // espera confirmación; si falla, reintenta
```

Si no llega la confirmación, se reenvía. Puede duplicar, nunca pierde. **Es lo que usa el 99% de los sistemas serios.**

### Exactly-once — lo que realmente significa

> [!warning] La distinción clave: entrega vs procesamiento
> - **Entrega exactamente una vez**: imposible (dos generales).
> - **Procesamiento con efecto exactamente una vez**: alcanzable — se construye con `at-least-once` + idempotencia del consumidor.
>
> Cuando alguien dice "tenemos exactly-once", o está hablando del efecto, o está equivocado.

```
at-least-once (transporte)  +  idempotencia (consumidor)  =  efecto exactamente una vez
```

Ver [[03 - Idempotencia y Deduplicacion]] para el lado del consumidor.

## 🔧 Kafka en concreto

### Productor

```yaml
spring:
  kafka:
    producer:
      acks: all                                  # todas las réplicas en sincronía
      retries: 2147483647
      properties:
        enable.idempotence: true                 # evita duplicados por reintento
        max.in.flight.requests.per.connection: 5 # con idempotencia, preserva el orden
        delivery.timeout.ms: 120000
```

| `acks` | Espera | Riesgo |
|---|---|---|
| `0` | nada | se pierde si el líder cae |
| `1` | solo el líder | se pierde si el líder cae antes de replicar |
| `all` | todas las réplicas en sincronía | ninguno mientras quede una réplica |

> [!tip] `enable.idempotence: true` resuelve el duplicado **del productor**
> El productor numera cada mensaje con un identificador de sesión y un número de secuencia; el broker descarta los repetidos. Elimina los duplicados causados por reintentos del productor — pero **no** los que ve el consumidor cuando se reprocesa un offset.

### Consumidor

```yaml
spring:
  kafka:
    consumer:
      enable-auto-commit: false        # el commit lo controlamos nosotros
      auto-offset-reset: earliest
      isolation-level: read_committed   # no lee mensajes de transacciones abiertas
    listener:
      ack-mode: manual
```

El orden de las operaciones determina la garantía:

```java
// ❌ AT-MOST-ONCE: si falla el procesamiento, el mensaje ya se dio por consumido
public void alRecibir(Evento e, Acknowledgment ack) {
    ack.acknowledge();
    procesar(e);            // si esto falla, el evento se perdió
}

// ✅ AT-LEAST-ONCE: se confirma después de procesar
public void alRecibir(Evento e, Acknowledgment ack) {
    procesar(e);
    ack.acknowledge();      // si el proceso muere antes, se reentrega
}
```

> [!danger] Confirmar antes de procesar convierte silenciosamente el sistema en at-most-once
> Es un error de una línea con consecuencias graves: los eventos se pierden sin dejar rastro. Nadie se entera hasta que falta un dato en un reporte semanas después.

### Las transacciones de Kafka

```java
@Transactional("kafkaTransactionManager")
public void procesarYReemitir(Evento entrada) {
    var salida = transformar(entrada);
    kafkaTemplate.send("procesados", salida);
    // el offset de entrada y el mensaje de salida se confirman atómicamente
}
```

> [!warning] La atomicidad vale solo dentro de Kafka
> El patrón *consume-transform-produce* sí es atómico: leer de un topic, transformar y escribir en otro. Pero si en el medio escribís en Postgres o llamás a una API externa, **eso queda fuera de la transacción**. Volvés al problema del dual-write, que se resuelve con [[05 - Transactional Outbox]].

## 🔢 El orden

Kafka garantiza orden **dentro de una partición**, no dentro de un topic.

```java
// La clave determina la partición: mismos eventos de la misma cuenta, misma partición
kafkaTemplate.send("movimientos", cuentaId.toString(), evento);
```

```
Topic "movimientos", 3 particiones

Partición 0: [ACC-1 dep] [ACC-1 ret] [ACC-1 transf]   ← ordenado ✅
Partición 1: [ACC-2 dep] [ACC-2 transf]               ← ordenado ✅
Partición 2: [ACC-3 dep]

Entre particiones: NO hay orden garantizado.
```

> [!tip] La clave de partición es una decisión de diseño, no un detalle
> Si necesitás que los eventos de una cuenta se procesen en orden, la clave debe ser la cuenta. Con clave aleatoria (o nula) los eventos se reparten y el orden se pierde — y un retiro puede procesarse antes que el depósito que lo financia.

| Clave | Consecuencia |
|---|---|
| `null` | reparto rotativo: máximo paralelismo, cero orden |
| `cuentaId` | orden por cuenta ✅ |
| valor constante | todo en una partición: orden total, cero paralelismo |
| valor de baja cardinalidad | particiones desbalanceadas (*hot partition*) |

> [!warning] El orden y el paralelismo son un intercambio
> No podés tener orden total y procesamiento paralelo al mismo tiempo. La granularidad correcta suele ser "orden por agregado": todo lo de una cuenta ordenado, cuentas distintas en paralelo.

## 💀 Cuando el mensaje no se puede procesar

```java
@Bean
public DefaultErrorHandler manejadorDeErrores(KafkaTemplate<String, Object> template) {
    var recuperador = new DeadLetterPublishingRecoverer(template,
        (registro, ex) -> new TopicPartition(registro.topic() + ".DLT", registro.partition()));

    var backoff = new ExponentialBackOffWithMaxRetries(3);
    backoff.setInitialInterval(1000);
    backoff.setMultiplier(2.0);

    var manejador = new DefaultErrorHandler(recuperador, backoff);
    manejador.addNotRetryableExceptions(EventoMalFormadoException.class);
    return manejador;
}
```

| Tipo de fallo | Acción |
|---|---|
| Transitorio (red, dependencia caída) | reintentar con backoff |
| Permanente (payload inválido) | a la DLT, sin reintentar |
| Envenenado (rompe siempre) | DLT + alerta |

> [!danger] Un mensaje envenenado sin DLT bloquea la partición entera
> Kafka entrega en orden dentro de la partición: si el consumidor no confirma el offset, se queda ahí reintentando para siempre y **ningún mensaje posterior de esa partición se procesa**. Un solo mensaje mal formado puede detener el procesamiento de miles de cuentas.

> [!warning] Una DLT sin monitoreo no sirve
> Los mensajes se acumulan en silencio. Hace falta métrica de profundidad y alerta, más un procedimiento documentado de reproceso.

## 📊 Elegir la garantía

| Caso de uso | Garantía | Por qué |
|---|---|---|
| Métricas y telemetría | at-most-once | perder muestras es tolerable |
| Notificación al cliente | at-least-once | un correo repetido molesta, uno perdido es peor |
| Movimiento de dinero | at-least-once + idempotencia | ni perder ni duplicar |
| Proyección de saldos | at-least-once + idempotencia | debe converger al valor correcto |
| Auditoría | at-least-once + dedup | perder es inaceptable |

## 🧪 Tests

```java
@Test
void un_fallo_al_procesar_no_confirma_el_offset() {
    doThrow(new RuntimeException()).when(manejador).procesar(any());
    assertThrows(RuntimeException.class, () -> consumidor.alRecibir(evento, ack));
    verify(ack, never()).acknowledge();          // se reentregará
}

@Test
void los_eventos_de_una_cuenta_llegan_en_orden() {
    IntStream.range(0, 100).forEach(i -> publicar(cuentaId, eventoNumero(i)));
    assertEquals(ordenados(0, 100), consumidor.recibidosPara(cuentaId));
}

@Test
void un_mensaje_mal_formado_va_a_la_DLT_sin_reintentos() {
    publicar(topic, mensajeInvalido());
    esperarQue(() -> contarEn(topic + ".DLT") == 1);
    verify(manejador, times(1)).procesar(any());   // no reintentó
}

@Test
void un_mensaje_envenenado_no_bloquea_los_siguientes() {
    publicar(topic, mensajeInvalido());
    publicar(topic, mensajeValido());
    esperarQue(() -> procesados.contains(mensajeValido()));
}
```

Con Testcontainers y `KafkaContainer` esto se prueba de verdad, no con mocks.

## ⚠️ Errores Comunes

> [!warning] Los que cuestan datos
> - **Confirmar el offset antes de procesar**: at-most-once accidental.
> - **`acks: 1`** en eventos de negocio: se pierden si cae el líder.
> - **Creer que las transacciones de Kafka cubren la base de datos**: no lo hacen.
> - **Clave de partición nula o aleatoria** cuando el orden importa.
> - **Sin DLT**: un mensaje envenenado bloquea la partición.
> - **DLT sin monitoreo**: los mensajes mueren en silencio.
> - **Reintentar errores deterministas**: nunca van a pasar.
> - **`auto-commit: true`**: confirma por tiempo, no por procesamiento; pierde y duplica de forma impredecible.

## 📌 Notas Rápidas
- La frase que hay que poder decir en una entrevista: *"exactly-once no existe en el transporte; se construye con at-least-once más idempotencia en el consumidor"*. Distingue a quien entendió el problema de quien repite el marketing de un producto.
- `auto-commit: true` es el default de Kafka y es la peor configuración posible para eventos de negocio: confirma offsets por temporizador, sin relación con si el procesamiento terminó.
- El problema de los dos generales se enunció en 1975 y sigue siendo la razón por la que existe toda esta nota. No es un problema de ingeniería: es una imposibilidad demostrada.

## 🔗 Relaciones
- **Módulo**: [[Sistemas Distribuidos]]
- **Anterior**: [[03 - Idempotencia y Deduplicacion]]
- **Siguiente**: [[05 - Transactional Outbox]]
- **Se conecta con**: [[01 - Kafka Fundamentos]], [[02 - Kafka con Spring Boot]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

