---
tags:
  - Java
  - Proyecto
  - Banca
  - corebank
Creado: 2026-09-08
Relacionado:
  - Proyectos Java
  - Banca y Fintech
  - Bases de Datos Java
Palabras clave:
  - idempotencia, Idempotency-Key, transacciones, bloqueo optimista, reversa, concurrencia
Estado:
  - Pendiente
Nivel: Intermedio-Avanzado
Prerequisitos: Fase 1, Spring Data, Bases de Datos Java
---

# Fase 2 · Transferencias e idempotencia

## 🎯 Objetivo

La fase donde `corebank` deja de ser un CRUD y se convierte en un sistema financiero. Se agrega la transferencia — y con ella los tres problemas que definen el dominio: **concurrencia, duplicación y corrección de errores**.

> [!danger] Esta es la fase más importante del laboratorio
> Todo lo anterior lo sabe hacer cualquier backend. Lo de esta fase es lo que se pregunta en una entrevista de banca, y lo que más gente responde mal.

## 🧠 Los tres problemas

### Problema 1 — Dos transferencias al mismo tiempo

```
Saldo de ACC-001: S/ 100

Hilo A                          Hilo B
lee saldo → 100                 lee saldo → 100
valida 100 >= 80  ✅            valida 100 >= 80  ✅
registra -80                    registra -80
                                
Saldo final: S/ -60  💥  se retiró 160 de una cuenta con 100
```

Es una **condición de carrera de lectura-modificación-escritura**. La validación pasó en los dos hilos porque ambos leyeron antes de que el otro escribiera. `@Transactional` por sí solo **no** lo evita: el nivel de aislamiento por defecto (`READ COMMITTED`) permite exactamente esto.

### Problema 2 — El cliente reintenta

```
Cliente → POST /transferencias (S/ 500)
          ...timeout de red a los 30s...
Cliente → POST /transferencias (S/ 500)   ← reintenta, cree que falló

El servidor procesó las DOS. Se transfirieron S/ 1000.
```

El cliente nunca supo si la primera llegó. Y la respuesta correcta no es "que no reintente": los reintentos son inevitables. La solución es que **repetir la misma operación no cambie el resultado**: idempotencia.

### Problema 3 — Se transfirió mal

Un operador transfiere S/ 5.000 a la cuenta equivocada. La tentación es `DELETE` o `UPDATE`. En un sistema financiero eso es inaceptable: destruye la traza y ningún auditor lo acepta. Se corrige con un **asiento de compensación** que deja los dos hechos registrados.

## 🛡️ Solución 1 — Idempotencia

El cliente manda una clave única por intención de operación. El servidor la registra con **restricción única**: la base garantiza que solo una ejecución gana.

```sql
-- V2__idempotencia.sql
CREATE TABLE idempotencia (
    clave           VARCHAR(64)  PRIMARY KEY,
    huella_request  VARCHAR(64)  NOT NULL,   -- hash del body
    estado          VARCHAR(20)  NOT NULL,   -- EN_PROCESO | COMPLETADA
    respuesta       JSONB,
    asiento_id      UUID,
    creado_en       TIMESTAMPTZ  NOT NULL DEFAULT now(),
    expira_en       TIMESTAMPTZ  NOT NULL
);

CREATE INDEX idx_idempotencia_expira ON idempotencia(expira_en);
```

```java
@Service
public class ServicioIdempotencia {

    private final IdempotenciaRepository repo;
    private final ObjectMapper mapper;

    /**
     * Ejecuta la operación una sola vez por clave.
     * Si la clave ya existe: devuelve la respuesta guardada, no reejecuta.
     */
    @Transactional
    public <T> T ejecutarUnaVez(String clave, Object request, Class<T> tipo,
                                Supplier<T> operacion) {

        String huella = hash(request);

        Optional<Idempotencia> previa = repo.findById(clave);
        if (previa.isPresent()) {
            Idempotencia registro = previa.get();

            // Misma clave con otro body = el cliente se equivocó
            if (!registro.huellaRequest().equals(huella)) {
                throw new ClaveIdempotenciaReutilizadaException(clave);
            }
            if (registro.estado() == EN_PROCESO) {
                throw new OperacionEnProcesoException(clave);   // 409, que reintente después
            }
            return leer(registro.respuesta(), tipo);            // ← respuesta original
        }

        try {
            repo.insertar(new Idempotencia(clave, huella, EN_PROCESO,
                                           Instant.now().plus(Duration.ofHours(24))));
        } catch (DataIntegrityViolationException e) {
            // Otro hilo insertó la misma clave entre el SELECT y el INSERT
            throw new OperacionEnProcesoException(clave);
        }

        T resultado = operacion.get();
        repo.completar(clave, escribir(resultado));
        return resultado;
    }
}
```

> [!tip] Por qué la restricción única y no un `if`
> Entre el `SELECT` que no encuentra la clave y el `INSERT` hay una ventana. Dos hilos pueden pasar los dos por el `if`. La **base** es el único árbitro confiable: uno de los dos `INSERT` falla con violación de unicidad, y ese es el mecanismo de exclusión.

## 🔒 Solución 2 — Concurrencia

Dos estrategias. Para transferencias conviene el **bloqueo pesimista**, porque la contención sobre una misma cuenta es real y reintentar es caro.

```java
public interface CuentaRepository extends JpaRepository<CuentaEntity, UUID> {

    // Bloqueo pesimista: SELECT ... FOR UPDATE
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT c FROM CuentaEntity c WHERE c.id = :id")
    Optional<CuentaEntity> findByIdParaActualizar(@Param("id") UUID id);
}
```

```java
@Service
public class ServicioTransferencia {

    @Transactional
    public TransferenciaResponse transferir(TransferenciaRequest req) {

        // ⚠️ ORDEN FIJO: siempre se bloquea primero el UUID menor.
        // Sin esto, A→B y B→A simultáneas producen un DEADLOCK.
        UUID primero  = req.origen().compareTo(req.destino()) < 0 ? req.origen() : req.destino();
        UUID segundo  = primero.equals(req.origen()) ? req.destino() : req.origen();

        CuentaEntity c1 = cuentas.findByIdParaActualizar(primero)
            .orElseThrow(() -> new CuentaNoEncontradaException(primero));
        CuentaEntity c2 = cuentas.findByIdParaActualizar(segundo)
            .orElseThrow(() -> new CuentaNoEncontradaException(segundo));

        CuentaEntity origen  = c1.getId().equals(req.origen()) ? c1 : c2;
        CuentaEntity destino = origen == c1 ? c2 : c1;

        Dinero importe = new Dinero(req.monto(), origen.getMoneda());
        validar(origen, destino, importe);

        // Ahora el saldo leído es confiable: nadie más puede tocarlo
        Dinero disponible = libro.saldo(origen.getId(), importe.moneda());
        if (disponible.compareTo(importe) < 0) {
            throw new SaldoInsuficienteException(origen.getId(), disponible, importe);
        }

        Asiento asiento = Asiento.de("TRANSFERENCIA",
            new Apunte(origen.getId(),  importe.negado()),
            new Apunte(destino.getId(), importe));

        libro.registrar(asiento);
        return TransferenciaResponse.de(asiento);
    }
}
```

> [!danger] El deadlock de las transferencias cruzadas
> Si A→B bloquea A y después B, mientras B→A bloquea B y después A, ambas transacciones se esperan para siempre. Postgres mata una a los pocos segundos y el usuario ve un error aleatorio bajo carga.
> **Ordenar los bloqueos por un criterio determinista (el UUID) elimina el problema por completo.** Es de las cosas que más impresionan en una entrevista, porque casi nadie la menciona.

### Pesimista vs optimista

| | Pesimista (`FOR UPDATE`) | Optimista (`@Version`) |
|---|---|---|
| Cómo funciona | bloquea la fila hasta el commit | detecta el conflicto al escribir |
| Cuándo conviene | contención alta, reintentar es caro | contención baja, conflictos raros |
| Costo | espera y riesgo de deadlock | reintentos y `OptimisticLockException` |
| Para transferencias | ✅ | ⚠️ solo con poca contención |
| Para editar un perfil | ❌ innecesario | ✅ |

## ↩️ Solución 3 — Reversas

```java
@Transactional
public ReversaResponse reversar(UUID asientoId, String motivo, String usuario) {

    Asiento original = libro.buscar(asientoId)
        .orElseThrow(() -> new AsientoNoEncontradoException(asientoId));

    if (libro.tieneReversa(asientoId)) {
        throw new AsientoYaReversadoException(asientoId);   // una sola vez
    }

    // Se valida que el destino tenga fondos: la reversa puede fallar legítimamente
    validarFondosParaReversa(original);

    List<Apunte> invertidos = original.apuntes().stream()
        .map(ap -> new Apunte(ap.cuenta(), ap.importe().negado()))
        .toList();

    Asiento reversa = new Asiento(
        UUID.randomUUID(), Instant.now(),
        "REVERSA[%s] %s · por %s".formatted(asientoId, motivo, usuario),
        invertidos);

    libro.registrar(reversa);
    libro.vincularReversa(asientoId, reversa.id());
    return ReversaResponse.de(reversa);
}
```

> [!warning] La reversa puede fallar, y está bien
> Si transferiste S/ 5.000 por error y el destinatario ya gastó S/ 4.000, la reversa no puede ejecutarse. Un sistema que la fuerza deja la cuenta destino en negativo. Eso deja de ser un problema técnico y pasa a ser uno legal — el sistema debe rechazarla y escalar el caso.

## 🌐 El endpoint

```java
@PostMapping("/api/v1/transferencias")
@ResponseStatus(HttpStatus.CREATED)
public TransferenciaResponse transferir(
        @RequestHeader("Idempotency-Key")
        @Pattern(regexp = "^[A-Za-z0-9_-]{16,64}$") String clave,
        @Valid @RequestBody TransferenciaRequest request) {

    return idempotencia.ejecutarUnaVez(
        clave, request, TransferenciaResponse.class,
        () -> servicio.transferir(request));
}
```

| Situación | Código | Cuerpo |
|---|---|---|
| Primera vez, todo bien | `201` | la transferencia creada |
| Misma clave, mismo body | `200` | **la misma respuesta**, sin reejecutar |
| Misma clave, body distinto | `422` | clave reutilizada |
| Misma clave, aún en proceso | `409` | reintentar más tarde |
| Saldo insuficiente | `422` | disponible y solicitado |
| Cuenta inexistente | `404` | — |

## 🧪 Tests que definen la fase

```java
@Test
void dos_transferencias_simultaneas_no_dejan_saldo_negativo() throws Exception {
    crearCuentaCon("ACC-001", Dinero.de("100", PEN));

    var pool = Executors.newFixedThreadPool(2);
    var listos = new CountDownLatch(1);

    Callable<Boolean> intento = () -> {
        listos.await();                       // arrancan exactamente juntos
        try {
            servicio.transferir(new TransferenciaRequest(acc1, acc2, new BigDecimal("80")));
            return true;
        } catch (SaldoInsuficienteException e) {
            return false;
        }
    };

    var f1 = pool.submit(intento);
    var f2 = pool.submit(intento);
    listos.countDown();

    // Exactamente una debe pasar
    assertEquals(1, Stream.of(f1.get(), f2.get()).filter(b -> b).count());
    assertEquals(Dinero.de("20", PEN), libro.saldo(acc1, PEN));
}
```

```java
@Test
void el_mismo_request_repetido_transfiere_una_sola_vez() {
    var request = new TransferenciaRequest(acc1, acc2, new BigDecimal("500"));

    var r1 = api.transferir("clave-abc-123", request);
    var r2 = api.transferir("clave-abc-123", request);   // reintento

    assertEquals(r1.asientoId(), r2.asientoId());        // misma respuesta
    assertEquals(1, libro.contarPorMotivo("TRANSFERENCIA"));
    assertEquals(Dinero.de("500", PEN), libro.saldo(acc2, PEN));
}
```

```java
@Test
void transferencias_cruzadas_simultaneas_no_producen_deadlock() {
    // A→B y B→A al mismo tiempo, 100 veces. Ninguna debe fallar por deadlock.
    assertTimeoutPreemptively(Duration.ofSeconds(30), () -> { /* ... */ });
}
```

```java
@Test
void tras_cualquier_secuencia_de_operaciones_el_sistema_cuadra() {
    // La invariante de la Fase 0 sigue valiendo con base de datos y concurrencia
    assertEquals(0, libro.totalGlobal().signum());
}
```

## ✅ Criterios de aceptación

- [ ] `POST /transferencias` exige `Idempotency-Key` y rechaza el request sin ella.
- [ ] Repetir la misma clave con el mismo body devuelve la respuesta original sin reejecutar.
- [ ] Misma clave con distinto body devuelve `422`.
- [ ] La unicidad de la clave está garantizada por la base, no por código Java.
- [ ] Los bloqueos se toman en orden determinista y hay un test que prueba ausencia de deadlock.
- [ ] Un test concurrente demuestra que el saldo nunca queda negativo.
- [ ] La reversa genera un asiento nuevo; el original permanece intacto.
- [ ] Un asiento no puede reversarse dos veces.
- [ ] El test de cuadre global sigue pasando.
- [ ] Existe un job que limpia claves de idempotencia vencidas.

## ⚠️ Trampas de esta fase

> [!warning] Las cinco clásicas
> - **Creer que `@Transactional` resuelve la concurrencia.** Da atomicidad, no exclusión mutua. Necesitás además un bloqueo.
> - **Bloquear las cuentas en el orden del request.** Es exactamente la receta del deadlock.
> - **Generar la clave de idempotencia en el servidor.** Entonces cada reintento trae una clave nueva y no sirve para nada: la genera el **cliente**, una por intención.
> - **Guardar la clave sin restricción única.** El `if` en Java no protege contra dos hilos.
> - **Reversar con `DELETE`.** Destruye la traza de auditoría. En banca eso es inadmisible.

## 🔁 Retrospectiva

- ¿Por qué la restricción única de base es más confiable que un `if` en Java?
- ¿Elegiste bloqueo pesimista u optimista? ¿Qué te haría cambiar de opinión?
- ¿Qué pasa si el servidor muere justo después de registrar el asiento pero antes de marcar la clave como completada?
- ¿Cuánto tiempo debería vivir una clave de idempotencia, y por qué?

## 🔗 Relaciones
- **Anterior**: [[Fase 1 - API de Cuentas]]
- **Siguiente**: [[Fase 3 - Seguridad y Auditoria]]
- **Teoría que aplica**: [[05 - Idempotencia Financiera]], [[03 - Idempotencia y Deduplicacion]], [[05 - Transacciones]], [[10 - Locks y Bloqueo Optimista]]
- **Índice**: [[Proyectos Java]]

## 📌 Notas Rápidas
- La pregunta del retrospectiva sobre "el servidor muere entre el asiento y el commit" tiene truco: si ambas escrituras están en **la misma transacción**, no puede pasar. Si están en transacciones distintas, sí — y eso es el problema del dual-write que se resuelve en la [[Fase 5 - Eventos y Outbox]].
- `Idempotency-Key` es un header estándar de facto (lo usan Stripe, Adyen y la mayoría de las pasarelas). Vale la pena leer cómo lo documenta Stripe.
- Si en una entrevista te preguntan cómo evitar transferencias duplicadas, la respuesta completa es: clave de idempotencia generada por el cliente + restricción única en base + respuesta cacheada. Decir solo "uso transacciones" es la respuesta incorrecta.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

