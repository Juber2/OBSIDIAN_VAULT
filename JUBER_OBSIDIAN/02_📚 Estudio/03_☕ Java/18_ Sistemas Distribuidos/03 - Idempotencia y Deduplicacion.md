---
tags:
  - Java
  - Distribuido
  - Patrones
Creado: 2026-09-08
Relacionado:
  - Sistemas Distribuidos
  - 05 - Idempotencia Financiera
Palabras clave:
  - idempotencia, deduplicacion, inbox, ventana de dedup, clave natural, reintento, backoff
---

## 🧠 Concepto

Esta nota es la contraparte teórica de [[05 - Idempotencia Financiera]]. Aquella cubre el caso HTTP y bancario; esta cubre **por qué la idempotencia es estructural** en un sistema distribuido, y cómo se implementa del lado del consumidor de eventos.

La cadena de razonamiento es corta y sin salida:

```
La red no es confiable          ([[01 - Falacias de la Computacion Distribuida]])
        ↓
Hay que reintentar
        ↓
Reintentar duplica efectos
        ↓
La única salida: que repetir NO tenga efecto adicional
```

## 🔢 Idempotencia, formalmente

Una operación `f` es idempotente si `f(f(x)) = f(x)`.

| Operación | ¿Idempotente? | Por qué |
|---|---|---|
| `saldo = 1000` | ✅ | asignación absoluta |
| `saldo += 100` | ❌ | acumula |
| `estado = COMPLETADA` | ✅ | asignación |
| `contador++` | ❌ | acumula |
| `DELETE WHERE id = x` | ✅ | ya borrado sigue borrado |
| `INSERT` con clave única | ✅ | el segundo falla |
| `INSERT` sin restricción | ❌ | crea duplicados |
| enviar un correo | ❌ | efecto externo observable |

> [!tip] Preferí asignaciones absolutas sobre incrementos
> `saldo = 1000` se puede repetir sin daño; `saldo += 100` no. Cuando el diseño lo permite, expresar la operación como "el estado final es X" en vez de "sumá X" elimina el problema de raíz.

Los métodos HTTP `GET`, `PUT` y `DELETE` son idempotentes **por especificación**. `POST` no lo es, y por eso necesita un mecanismo explícito.

## 🔁 Reintentos: cuándo y cómo

```java
@Bean
public Retry politicaDeReintentos() {
    return Retry.of("servicioExterno", RetryConfig.custom()
        .maxAttempts(3)
        .intervalFunction(IntervalFunction.ofExponentialRandomBackoff(
            Duration.ofMillis(200),   // inicial
            2.0,                       // multiplicador
            0.5))                      // jitter: ±50%
        .retryExceptions(IOException.class, TimeoutException.class)
        .ignoreExceptions(ExcepcionDeNegocio.class)     // determinista: no reintentar
        .build());
}
```

> [!danger] El jitter no es un detalle: evita la tormenta de reintentos
> Sin aleatoriedad, todos los clientes que fallaron en el mismo instante reintentan exactamente al mismo tiempo. Eso genera picos sincronizados que tumban el servicio justo cuando se está recuperando — el *thundering herd*. El jitter dispersa los reintentos.

### Qué se reintenta y qué no

| Error | ¿Reintentar? |
|---|---|
| Timeout de conexión | ✅ probablemente no llegó |
| `503` / `429` (con `Retry-After`) | ✅ transitorio |
| Timeout de lectura | ⚠️ solo si es idempotente — **pudo haberse ejecutado** |
| `400` / `422` | ❌ determinista |
| `401` / `403` | ❌ no cambia reintentando |
| `404` | ❌ |
| `409` conflicto | ⚠️ depende |

> [!warning] El timeout de lectura es el caso peligroso
> Un timeout de **conexión** significa que no llegó. Un timeout de **lectura** significa que llegó y no sabés qué pasó. Reintentar el segundo sin idempotencia es exactamente cómo se duplican las operaciones.

## 📥 Deduplicación del lado del consumidor

Todo broker realista entrega **al menos una vez** (ver [[04 - Garantias de Entrega]]). El consumidor tiene que tolerar duplicados.

### El patrón Inbox

```sql
CREATE TABLE eventos_procesados (
    evento_id     UUID        NOT NULL,
    consumidor    VARCHAR(60) NOT NULL,
    procesado_en  TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (evento_id, consumidor)      -- ← la base es el árbitro
);

CREATE INDEX idx_procesados_limpieza ON eventos_procesados(procesado_en);
```

```java
@KafkaListener(topics = "movimientos", groupId = "notificaciones")
@Transactional
public void alRecibir(MovimientoRegistrado evento, Acknowledgment ack) {

    try {
        procesados.registrar(evento.id(), "notificaciones");
    } catch (DataIntegrityViolationException e) {
        log.debug("evento {} ya procesado, se ignora", evento.id());
        ack.acknowledge();
        return;
    }

    manejador.procesar(evento);     // en la MISMA transacción que el registro
    ack.acknowledge();
}
```

> [!danger] El registro y el procesamiento van en la misma transacción
> Si se registra el evento y después el procesamiento falla, la transacción hace rollback y **también se deshace el registro**: el reintento vuelve a procesarlo. Si estuvieran en transacciones separadas, un fallo intermedio marcaría como procesado algo que no se procesó — y el evento se perdería para siempre.

> [!warning] La clave primaria incluye el consumidor
> Varios grupos de consumidores procesan el mismo evento por motivos distintos: notificaciones, antifraude, proyecciones. Cada uno lleva su propio registro. Sin el consumidor en la clave, el primero que procesa bloquea a todos los demás.

### La ventana de deduplicación

La tabla no puede crecer para siempre.

```java
@Scheduled(cron = "0 0 4 * * *")
public void limpiar() {
    procesados.borrarAnterioresA(Instant.now().minus(VENTANA));
}
```

| Ventana | Riesgo |
|---|---|
| 1 hora | un reintento tardío del broker se reprocesa |
| **7-30 días** | rango habitual |
| Infinita | la tabla crece sin límite |

> [!tip] La ventana debe superar el peor caso de reentrega
> Si el broker retiene mensajes 7 días y un consumidor puede estar caído todo ese tiempo, la ventana tiene que ser mayor a 7 días. Dimensionarla por debajo de la retención del broker deja un agujero.

## 🔑 Claves naturales

Antes de agregar una tabla de deduplicación, buscá si el dato ya tiene una identidad única:

```sql
-- Un evento por asiento y tipo: no hace falta tabla aparte
CREATE UNIQUE INDEX idx_notificacion ON notificacion(asiento_id, tipo);

-- Una liquidación por referencia del adquirente
CREATE UNIQUE INDEX idx_liquidacion ON liquidacion(referencia_externa);

-- Una proyección por versión del agregado
CREATE UNIQUE INDEX idx_proyeccion ON proyeccion(agregado_id, version);
```

> [!tip] La clave natural es siempre mejor
> No necesita tabla adicional, no necesita limpieza y la unicidad es una propiedad del dominio, no una capa técnica encima. La tabla de deduplicación es el recurso para cuando **no existe** clave natural.

## 🧩 Operaciones naturalmente idempotentes

A veces se puede rediseñar la operación para que el problema desaparezca:

```java
// ❌ Acumulativa: repetir suma de más
public void registrarPago(UUID facturaId, Dinero monto) {
    factura.pagado = factura.pagado.mas(monto);
}

// ✅ Absoluta: repetir da el mismo resultado
public void registrarPago(UUID facturaId, UUID pagoId, Dinero monto) {
    pagos.insertarSiNoExiste(pagoId, facturaId, monto);   // clave única
    factura.pagado = pagos.sumaDe(facturaId);             // derivado
}
```

Es el mismo principio del ledger de [[03 - Ledger y Partida Doble]]: **guardar los hechos y derivar el agregado** en vez de mutar un acumulador.

### Idempotencia con estado

Muchas transiciones se vuelven idempotentes con un control de estado:

```java
@Transactional
public void completar(UUID pedidoId) {
    Pedido pedido = repo.findByIdParaActualizar(pedidoId).orElseThrow();

    if (pedido.estado() == COMPLETADO) {
        return;                        // ya está: no es error, es idempotencia
    }
    if (pedido.estado() != EN_PROCESO) {
        throw new TransicionInvalidaException(pedido.estado(), COMPLETADO);
    }

    pedido.completar();
    eventos.publicar(new PedidoCompletado(pedidoId));
}
```

> [!warning] Repetir una operación ya aplicada no es un error
> Devolver `409` cuando el estado ya es el deseado obliga al llamante a distinguir "falló" de "ya estaba hecho". Si el resultado final es el que se pedía, la respuesta correcta es éxito.

## ⚠️ El límite: los efectos externos

```java
public void notificar(MovimientoRegistrado evento) {
    procesados.registrar(evento.id(), "notificaciones");   // transaccional ✅
    emailService.enviar(...);                              // externo ❌ no transaccional
}
```

Si el correo se envía y después la transacción falla, **el correo ya salió**. No hay rollback posible sobre un efecto externo.

Mitigaciones:

| Estrategia | Cómo |
|---|---|
| Efecto externo al final | reduce la ventana, no la elimina |
| Idempotencia en el proveedor | muchos aceptan una clave (`Idempotency-Key` en SendGrid, por ejemplo) |
| Outbox para el envío | el efecto externo lo dispara un proceso aparte, tras el commit |
| Aceptar el duplicado | un correo repetido molesta; un cobro repetido no se acepta |

> [!tip] Clasificá los efectos por su costo de duplicación
> Un correo duplicado es una molestia tolerable. Un cobro duplicado no lo es. El esfuerzo de ingeniería debe ser proporcional al daño: para el correo alcanza con el Outbox; para el cobro, idempotencia del lado del proveedor.

## 🧪 Tests

```java
@Test
void procesar_el_mismo_evento_dos_veces_produce_un_solo_efecto() {
    consumidor.alRecibir(evento, ack);
    consumidor.alRecibir(evento, ack);
    verify(manejador, times(1)).procesar(any());
}

@Test
void si_el_procesamiento_falla_el_evento_no_queda_marcado() {
    doThrow(new RuntimeException()).when(manejador).procesar(any());
    assertThrows(RuntimeException.class, () -> consumidor.alRecibir(evento, ack));
    assertFalse(procesados.existe(evento.id(), "notificaciones"));   // ← rollback
}

@Test
void dos_consumidores_distintos_procesan_el_mismo_evento() {
    notificaciones.alRecibir(evento, ack);
    antifraude.alRecibir(evento, ack);
    verify(manejadorNotif, times(1)).procesar(any());
    verify(manejadorFraude, times(1)).procesar(any());
}

@Test
void completar_un_pedido_ya_completado_no_falla_ni_reemite_evento() {
    servicio.completar(id);
    servicio.completar(id);
    verify(eventos, times(1)).publicar(any(PedidoCompletado.class));
}

@Test
void los_reintentos_usan_backoff_con_jitter() {
    // los intervalos no son todos iguales
}
```

## ⚠️ Errores Comunes

> [!warning] Los que producen duplicados en producción
> - **Registrar el evento y procesarlo en transacciones separadas**: se pierden eventos.
> - **Clave de deduplicación sin el consumidor**: el primero bloquea a los demás.
> - **Reintentar sobre timeout de lectura** en operaciones no idempotentes.
> - **Reintentos sin jitter**: tormenta sincronizada.
> - **Reintentar errores deterministas**: carga inútil.
> - **Deduplicación en memoria o en caché volátil**: se pierde al reiniciar y no se comparte entre instancias.
> - **Ventana de dedup menor a la retención del broker.**
> - **Devolver error cuando la operación ya estaba aplicada.**

## 📌 Notas Rápidas
- Si tuvieras que resumir el módulo en una frase: **"al menos una vez" es lo que la red te da; "exactamente una vez" es lo que vos construís encima, con idempotencia**.
- Toda tabla de deduplicación necesita tres cosas: clave primaria compuesta con el consumidor, participación en la transacción del procesamiento, y limpieza programada. Falta alguna en la mayoría de las implementaciones que vas a ver.
- Rediseñar la operación para que sea naturalmente idempotente siempre gana sobre agregar una capa de deduplicación. Preguntate primero si podés expresarla como asignación absoluta.

## 🔗 Relaciones
- **Módulo**: [[Sistemas Distribuidos]]
- **Anterior**: [[02 - CAP PACELC y Consistencia]]
- **Siguiente**: [[04 - Garantias de Entrega]]
- **Aplicación financiera**: [[05 - Idempotencia Financiera]]
- **Se implementa en**: [[Fase 5 - Eventos y Outbox]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

