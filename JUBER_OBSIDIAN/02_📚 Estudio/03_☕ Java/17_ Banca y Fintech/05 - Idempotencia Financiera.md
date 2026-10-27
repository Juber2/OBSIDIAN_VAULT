---
tags:
  - Java
  - Banca
  - Dominio
Creado: 2026-09-08
Relacionado:
  - Banca y Fintech
  - 03 - Idempotencia y Deduplicacion
Palabras clave:
  - idempotencia, Idempotency-Key, reintento, deduplicacion, huella de request, TTL
---

## 🧠 Concepto

Una operación es **idempotente** cuando ejecutarla N veces produce el mismo resultado que ejecutarla una vez.

Consultar un saldo es idempotente por naturaleza. Transferir dinero **no lo es** — y ahí está el problema, porque en una red los reintentos son inevitables.

## 💥 El problema, en concreto

```
Cliente ──POST /transferencias {S/ 500}──► Servidor
                                              │ procesa ✅
                                              │ registra el asiento
        ◄──────────── ✂️ se corta la red ─────┘

El cliente ve un timeout. No sabe si:
  a) la petición nunca llegó
  b) llegó, se procesó, y se perdió la respuesta

Reintenta. Se transfieren S/ 1.000.
```

> [!danger] El cliente no puede distinguir los dos casos
> Y no hay forma de que pueda: un timeout es indistinguible de un fallo. Decir "que no reintente" no es una solución — la app móvil reintenta, el balanceador reintenta, Feign reintenta. **El servidor tiene que tolerar el reintento.**

## 🔑 La clave de idempotencia

El cliente genera un identificador único **por intención de operación** y lo manda en cada intento. El servidor lo usa para reconocer que ya vio esa operación.

```
POST /api/v1/transferencias
Idempotency-Key: 7f3a9b2c-4d5e-6f70-8192-a3b4c5d6e7f8
Content-Type: application/json

{ "origen": "ACC-001", "destino": "ACC-002", "monto": 500.00 }
```

> [!warning] La genera el cliente, no el servidor
> Si la generara el servidor, cada reintento traería una clave nueva y no serviría para nada. La clave identifica **la intención del usuario**, y esa intención nace en el cliente.
> Regla práctica: se genera cuando el usuario aprieta el botón, no cuando se envía la petición.

### Una clave por intención, no por petición

```java
// ❌ una clave nueva en cada reintento: inútil
public void transferir(Req req) {
    http.post(url, req, Map.of("Idempotency-Key", UUID.randomUUID().toString()));
}

// ✅ la clave se genera una vez y se reusa en los reintentos
public void transferir(Req req) {
    String clave = UUID.randomUUID().toString();
    reintentar(3, () -> http.post(url, req, Map.of("Idempotency-Key", clave)));
}
```

## 🗄️ La implementación

```sql
CREATE TABLE idempotencia (
    clave           VARCHAR(64)  PRIMARY KEY,     -- ← la unicidad la impone la BASE
    huella_request  VARCHAR(64)  NOT NULL,        -- SHA-256 del body
    endpoint        VARCHAR(120) NOT NULL,        -- alcance
    usuario_id      UUID,                         -- alcance
    estado          VARCHAR(20)  NOT NULL,        -- EN_PROCESO | COMPLETADA | FALLIDA
    codigo_http     SMALLINT,
    respuesta       JSONB,
    creado_en       TIMESTAMPTZ  NOT NULL DEFAULT now(),
    expira_en       TIMESTAMPTZ  NOT NULL
);

CREATE INDEX idx_idempotencia_expira ON idempotencia(expira_en);
```

```java
@Transactional
public <T> ResultadoIdempotente<T> ejecutarUnaVez(
        String clave, Object request, Class<T> tipo, Supplier<T> operacion) {

    String huella = sha256(request);

    // 1 · ¿Ya la vimos?
    Optional<Idempotencia> previa = repo.findById(clave);
    if (previa.isPresent()) {
        return resolverExistente(previa.get(), huella, tipo);
    }

    // 2 · Reservar la clave. La restricción única es el árbitro.
    try {
        repo.insertar(Idempotencia.enProceso(clave, huella, endpointActual(),
                                             usuarioActual(), ttl()));
    } catch (DataIntegrityViolationException e) {
        // Otro hilo ganó la carrera entre el SELECT y el INSERT
        throw new OperacionEnProcesoException(clave);
    }

    // 3 · Ejecutar y guardar el resultado
    try {
        T resultado = operacion.get();
        repo.completar(clave, 201, serializar(resultado));
        return ResultadoIdempotente.nueva(resultado);

    } catch (ExcepcionDeNegocio e) {
        // Un rechazo determinista también se cachea: reintentar dará lo mismo
        repo.fallar(clave, e.codigoHttp(), serializar(e.aProblemDetail()));
        throw e;
    }
}
```

> [!tip] La restricción única de base es el único mecanismo confiable
> Entre el `SELECT` que no encuentra la clave y el `INSERT` hay una ventana en la que dos hilos pueden pasar los dos. Un `if` en Java no protege nada. La base sí: uno de los dos `INSERT` falla, y esa falla **es** el mecanismo de exclusión mutua.

## 🎭 Los cuatro escenarios

| Situación | Respuesta | Por qué |
|---|---|---|
| Clave nueva | `201` + resultado | primera ejecución |
| Misma clave, mismo body, completada | `200` + **la respuesta original** | no se reejecuta |
| Misma clave, body distinto | `422` clave reutilizada | error del cliente |
| Misma clave, aún en proceso | `409` + `Retry-After` | reintentar en unos segundos |

```java
private <T> ResultadoIdempotente<T> resolverExistente(
        Idempotencia registro, String huella, Class<T> tipo) {

    if (!registro.huellaRequest().equals(huella)) {
        throw new ClaveIdempotenciaReutilizadaException(registro.clave());
    }
    if (registro.estado() == EN_PROCESO) {
        throw new OperacionEnProcesoException(registro.clave());
    }
    return ResultadoIdempotente.repetida(deserializar(registro.respuesta(), tipo));
}
```

> [!warning] La huella del request no es opcional
> Sin ella, un cliente con un bug que reusa la misma clave para dos transferencias distintas recibiría la respuesta de la primera para la segunda — y la segunda **nunca se ejecutaría**. El dinero no se mueve y el cliente cree que sí. Comparar la huella convierte ese bug silencioso en un `422` visible.

## 🎯 El alcance de la clave

Una clave no vale globalmente: vale para **un endpoint y un usuario**.

```java
record ClaveIdempotencia(String valor, String endpoint, UUID usuarioId) {}
```

Sin alcance, dos clientes distintos que generen el mismo UUID (o un cliente que use claves predecibles como `1`, `2`, `3`) colisionan entre sí. Un usuario podría recibir la respuesta de otro — que es una fuga de datos, no solo un bug.

## ⏱️ Cuánto vive una clave

| TTL | Riesgo |
|---|---|
| Minutos | un reintento tardío se ejecuta de nuevo |
| **24 h – 7 días** | rango habitual en la industria |
| Para siempre | la tabla crece sin límite |

```java
@Scheduled(cron = "0 0 3 * * *")
public void limpiarVencidas() {
    int borradas = repo.borrarVencidas(Instant.now());
    log.atInfo().addKeyValue("borradas", borradas).log("limpieza de idempotencia");
}
```

> [!tip] El TTL lo define el negocio, no el técnico
> ¿Cuánto tiempo después puede un cliente reintentar legítimamente la misma transferencia? Si la app permite reintentar desde el historial 3 días después, el TTL tiene que cubrir eso.

## 🔐 Idempotencia natural

A veces no hace falta una clave explícita: la propia operación tiene un identificador único de negocio.

```sql
-- Un pago de una factura solo puede existir una vez
CREATE UNIQUE INDEX idx_pago_factura ON pago(factura_id);

-- Una liquidación de tarjeta llega con un identificador del adquirente
CREATE UNIQUE INDEX idx_liquidacion_ref ON liquidacion(referencia_adquirente);

-- Un asiento con referencia externa
CREATE UNIQUE INDEX idx_asiento_ref ON asiento(referencia) WHERE referencia IS NOT NULL;
```

> [!tip] Si existe una clave natural, usala
> Es más simple y más robusta que una tabla de idempotencia: la unicidad está en el dato mismo, no en una capa adicional. La clave explícita se reserva para operaciones que no tienen identificador natural.

## 🔁 Idempotencia vs deduplicación vs reintento

Tres cosas que se confunden:

| Concepto | Dónde vive | Qué resuelve |
|---|---|---|
| **Reintento** | el que llama | fallos transitorios |
| **Idempotencia** | el que recibe | que el reintento no duplique |
| **Deduplicación** | el consumidor de eventos | reentrega del broker |

```
Reintento sin idempotencia  = duplicar dinero
Idempotencia sin reintento  = fallos que no se recuperan
Los dos juntos              = resiliencia real
```

Ver [[04 - Garantias de Entrega]] para el lado de mensajería.

## ⚠️ El caso difícil: el proceso muere a mitad

```
1 · INSERT idempotencia (EN_PROCESO)   ✅
2 · registrar asiento                   ✅
3 · UPDATE idempotencia (COMPLETADA)    💥 el proceso muere
```

La clave queda en `EN_PROCESO` para siempre, y todo reintento recibe `409`.

**La solución correcta:** que los tres pasos estén en **la misma transacción**. Si el proceso muere, la transacción hace rollback completo y el reintento empieza limpio.

Si por alguna razón no pueden ir juntos, hace falta un recuperador:

```java
@Scheduled(fixedDelay = 60_000)
public void resolverAtascadas() {
    repo.enProcesoDesdeHace(Duration.ofMinutes(5)).forEach(reg -> {
        // ¿la operación llegó a ejecutarse?
        boolean ejecutada = asientos.existePorReferencia(reg.clave());
        if (ejecutada) repo.completarDesdeRecuperacion(reg.clave());
        else           repo.borrar(reg.clave());   // libera la clave
    });
}
```

## 🧪 Tests

```java
@Test
void el_mismo_request_repetido_transfiere_una_sola_vez() {
    var r1 = api.transferir("clave-1", request);
    var r2 = api.transferir("clave-1", request);

    assertEquals(r1.asientoId(), r2.asientoId());
    assertEquals(1, asientos.contarPorMotivo("TRANSFERENCIA"));
}

@Test
void misma_clave_con_otro_body_es_rechazada() {
    api.transferir("clave-2", requestDe500);
    assertThrows(ClaveIdempotenciaReutilizadaException.class,
        () -> api.transferir("clave-2", requestDe900));
}

@Test
void dos_peticiones_concurrentes_con_la_misma_clave_ejecutan_una_sola_vez() {
    // una responde 201, la otra 409 o 200 — nunca dos ejecuciones
}

@Test
void un_rechazo_de_negocio_tambien_se_cachea() {
    // reintentar una transferencia sin fondos no debe reejecutar la validación
}
```

## ⚠️ Errores Comunes

> [!warning] Los que anulan la protección
> - **Clave generada por el servidor**: cada reintento trae una distinta.
> - **Clave generada en cada envío** en vez de una por intención.
> - **Sin restricción única en base**: el `if` en Java no protege contra concurrencia.
> - **Sin huella del request**: un cliente con bug recibe la respuesta equivocada.
> - **Sin alcance por usuario**: colisión entre clientes = fuga de datos.
> - **Guardar la clave en caché en memoria**: se pierde al reiniciar y no se comparte entre instancias.
> - **No cachear los rechazos deterministas**: se reejecuta lógica que siempre dará el mismo error.

## 📌 Notas Rápidas
- `Idempotency-Key` es un estándar de facto: lo usan Stripe, Adyen, PayPal y prácticamente toda pasarela seria. Leer la documentación de Stripe sobre el tema es media hora bien invertida.
- Los métodos HTTP `GET`, `PUT` y `DELETE` son idempotentes **por definición** de la especificación. `POST` no lo es, y por eso necesita la clave.
- Redis no es el lugar para las claves de idempotencia de operaciones con dinero: es volátil, y si se pierde se duplican transferencias. La base transaccional es el lugar correcto.
- Se implementa en [[Fase 2 - Transferencias e Idempotencia]] y se vuelve obligatoria entre servicios en [[Fase 6 - Microservicios y SAGA]].

## 🔗 Relaciones
- **Módulo**: [[Banca y Fintech]]
- **Anterior**: [[04 - Cuentas Saldos y Movimientos]]
- **Siguiente**: [[06 - Transferencias y Consistencia]]
- **Teoría distribuida**: [[03 - Idempotencia y Deduplicacion]], [[04 - Garantias de Entrega]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

