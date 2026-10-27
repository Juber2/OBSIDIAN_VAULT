---
tags:
  - Java
  - Banca
  - Dominio
Creado: 2026-09-08
Relacionado:
  - Banca y Fintech
  - 05 - Idempotencia Financiera
Palabras clave:
  - transferencia, aislamiento, bloqueo, deadlock, estado desconocido, interbancaria, reversa
---

## 🧠 Concepto

Transferir es la operación más simple de enunciar y la más difícil de implementar bien: sacar de una cuenta y poner en otra, sin que el dinero se duplique ni se pierda, con múltiples peticiones simultáneas y con una red que falla.

## 🗺️ Los tres tipos

| Tipo | Alcance | Consistencia posible |
|---|---|---|
| **Interna** | dos cuentas del mismo banco, misma base | ✅ transacción ACID |
| **Interbancaria** | dos entidades distintas | ⚠️ eventual, con compensación |
| **Internacional** | cruza países y corresponsales | ⚠️ eventual, días de latencia |

La primera se resuelve con una transacción de base. Las otras dos **no tienen transacción posible** y necesitan SAGA.

## 🏠 Transferencia interna

Todo ocurre en una sola base, así que hay transacción real:

```java
@Transactional
public ResultadoTransferencia transferir(ComandoTransferir cmd) {

    if (cmd.origen().equals(cmd.destino()))
        throw new OperacionInvalidaException("origen y destino coinciden");

    // Bloqueo en orden determinista (ver más abajo)
    var cuentas = bloquearEnOrden(cmd.origen(), cmd.destino());
    Cuenta origen  = cuentas.origen();
    Cuenta destino = cuentas.destino();

    origen.exigirOperable();
    destino.exigirOperable();
    exigirMismaMoneda(origen, destino, cmd.importe());

    Dinero disponible = saldos.disponible(origen.id(), cmd.importe().moneda());
    if (disponible.compareTo(cmd.importe()) < 0)
        throw new SaldoInsuficienteException(origen.id(), disponible, cmd.importe());

    limites.verificar(origen, cmd.importe());

    Asiento asiento = Asiento.de("TRANSFERENCIA",
        new Apunte(origen.id(),  cmd.importe().negado()),
        new Apunte(destino.id(), cmd.importe()));

    libro.registrar(asiento);
    return ResultadoTransferencia.de(asiento);
}
```

Los dos apuntes están en el mismo asiento y la misma transacción: **o pasan los dos, o ninguno**. No existe el estado intermedio.

## 🔒 Concurrencia

### Por qué `@Transactional` no alcanza

```
Saldo: S/ 100          Hilo A                    Hilo B
                       lee saldo → 100
                                                 lee saldo → 100
                       valida 100 ≥ 80 ✅
                                                 valida 100 ≥ 80 ✅
                       escribe -80
                                                 escribe -80
Saldo final: -60 💥
```

Con el aislamiento por defecto (`READ COMMITTED`), esto pasa. La transacción garantiza **atomicidad**, no exclusión mutua.

### Niveles de aislamiento

| Nivel | Evita | Costo |
|---|---|---|
| `READ UNCOMMITTED` | nada | — |
| `READ COMMITTED` *(default Postgres)* | lecturas sucias | bajo |
| `REPEATABLE READ` | lecturas no repetibles | medio |
| `SERIALIZABLE` | todo | alto, con abortos por conflicto |

Subir a `SERIALIZABLE` resuelve el problema pero convierte los conflictos en excepciones que hay que reintentar, y degrada bastante bajo carga. Para transferencias, **el bloqueo explícito es más predecible**.

### Bloqueo pesimista

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT c FROM CuentaEntity c WHERE c.id = :id")
Optional<CuentaEntity> findByIdParaActualizar(@Param("id") UUID id);
```

Genera `SELECT ... FOR UPDATE`: la fila queda bloqueada hasta el commit. Cualquier otra transacción que la pida espera.

### El deadlock, y cómo eliminarlo

```
Transferencia A→B          Transferencia B→A
bloquea A ✅               bloquea B ✅
pide B ⏳ (la tiene B→A)   pide A ⏳ (la tiene A→B)
                  ☠️ deadlock
```

Postgres detecta el ciclo y mata una de las dos, pero el usuario ve un error aleatorio que solo aparece bajo carga — de los bugs más difíciles de reproducir.

```java
private ParDeCuentas bloquearEnOrden(CuentaId a, CuentaId b) {
    // SIEMPRE el UUID menor primero, sin importar quién es origen
    CuentaId primero = a.compareTo(b) < 0 ? a : b;
    CuentaId segundo = primero.equals(a) ? b : a;

    Cuenta c1 = repo.findByIdParaActualizar(primero).orElseThrow(...);
    Cuenta c2 = repo.findByIdParaActualizar(segundo).orElseThrow(...);

    return ParDeCuentas.de(c1, c2, a);   // reordena a origen/destino
}
```

> [!tip] Ordenar los bloqueos elimina el deadlock por completo
> No lo reduce ni lo hace menos probable: lo hace **imposible**. Si todas las transacciones piden los recursos en el mismo orden, no puede formarse un ciclo de espera. Es un resultado clásico de sistemas operativos y aplica igual acá.
> Es de las respuestas que más impresionan en una entrevista, porque casi nadie la menciona.

### Pesimista vs optimista

| | Pesimista | Optimista (`@Version`) |
|---|---|---|
| Mecanismo | bloquea y espera | detecta el conflicto al escribir |
| Conviene con | contención alta | conflictos raros |
| Costo | espera, riesgo de deadlock | reintentos |
| Transferencias | ✅ | ⚠️ solo con poca contención |

## 🌐 Transferencia interbancaria

Acá no hay base compartida ni transacción posible:

```
1 · reservar fondos en origen        (banco A)
2 · debitar origen                   (banco A)
3 · acreditar destino                (banco B) ← otra entidad, otra base
4 · confirmar                        (banco A)
```

Si el paso 3 falla, hay que **deshacer** los pasos 1 y 2 con operaciones inversas. Eso es una SAGA — desarrollada en [[06 - SAGA]] y aplicada en [[Fase 6 - Microservicios y SAGA]].

```java
public enum EstadoTransferencia {
    INICIADA, FONDOS_RESERVADOS, DEBITADA, ENVIADA,
    ACREDITADA, COMPLETADA,
    COMPENSANDO, FALLIDA, EN_REVISION      // ← intervención humana
}
```

> [!warning] Existe una ventana en la que el dinero no está en ningún lado
> Entre el débito en origen y la acreditación en destino, el dinero está "en tránsito": salió de una cuenta y no llegó a la otra. Contablemente vive en una **cuenta puente**.
> Esa ventana es inevitable y el negocio tiene que aceptarla explícitamente. Por eso las transferencias interbancarias muestran "en proceso" y no se acreditan al instante.

## ❓ El estado desconocido

El problema más difícil de todos: llamás al banco destino y hay timeout. **No sabés si acreditó**.

```java
public ResultadoAcreditacion acreditar(SolicitudAcreditacion solicitud) {
    try {
        return bancoExterno.acreditar(solicitud);

    } catch (TimeoutException e) {
        // ❌ NO compensar: podría haber acreditado
        // ❌ NO reintentar a ciegas: podría duplicar
        // ✅ consultar el estado real
        return consultarEstado(solicitud.referencia());
    }
}

private ResultadoAcreditacion consultarEstado(String referencia) {
    return bancoExterno.consultar(referencia)
        .map(ResultadoAcreditacion::desdeConsulta)
        .orElseGet(() -> {
            // Sigue sin saberse: marcar para revisión, no adivinar
            saga.pasarA(EN_REVISION);
            alertas.escalar("estado desconocido", referencia);
            return ResultadoAcreditacion.desconocido();
        });
}
```

> [!danger] Ante la duda, nunca adivines
> Compensar una operación que sí se ejecutó duplica el dinero al revés: el cliente recibe la plata en destino **y** se la devuelven en origen. Reintentar una que ya pasó la duplica hacia adelante.
> La única salida correcta es **consultar** (por eso toda operación externa necesita una referencia consultable) y, si sigue sin saberse, **escalar a un humano**. En banca eso es un procedimiento operativo normal, no un fallo del diseño.

## ↩️ Reversa, anulación y devolución

Tres cosas distintas que se confunden:

| Concepto | Cuándo | Efecto contable |
|---|---|---|
| **Anulación** | la operación aún no se liquidó | se cancela, no genera asiento |
| **Reversa** | ya se liquidó, fue un error | asiento inverso |
| **Devolución** | el destino la rechaza y la devuelve | asiento nuevo en sentido contrario |

```java
public Asiento reversar(UUID asientoId, String motivo, String operador) {
    Asiento original = libro.buscar(asientoId).orElseThrow();

    if (libro.tieneReversa(asientoId))
        throw new AsientoYaReversadoException(asientoId);

    // La reversa puede fallar legítimamente
    validarFondosParaReversa(original);

    List<Apunte> invertidos = original.apuntes().stream()
        .map(ap -> new Apunte(ap.cuenta(), ap.importe().negado()))
        .toList();

    return libro.registrar(new Asiento(UUID.randomUUID(), Instant.now(),
        "REVERSA[%s] %s · %s".formatted(asientoId, motivo, operador), invertidos));
}
```

> [!warning] La reversa puede ser imposible
> Si transferiste S/ 5.000 por error y el destinatario ya gastó S/ 4.000, no hay reversa técnica: dejaría la cuenta destino en descubierto. El sistema debe **rechazarla** y escalar el caso.
> Forzarla convierte un problema operativo en uno legal.

## 🧪 Tests

```java
@Test
void dos_transferencias_simultaneas_no_dejan_saldo_negativo() {
    crearCuentaCon(acc1, Dinero.de("100", PEN));
    var listos = new CountDownLatch(1);

    Callable<Boolean> intento = () -> {
        listos.await();
        try { servicio.transferir(cmd(acc1, acc2, "80")); return true; }
        catch (SaldoInsuficienteException e) { return false; }
    };

    var f1 = pool.submit(intento);
    var f2 = pool.submit(intento);
    listos.countDown();

    assertEquals(1, Stream.of(f1.get(), f2.get()).filter(b -> b).count());
    assertEquals(Dinero.de("20", PEN), saldos.contable(acc1, PEN));
}

@Test
void transferencias_cruzadas_simultaneas_no_producen_deadlock() {
    // A→B y B→A, 100 veces en paralelo, sin excepciones de deadlock
}

@Test
void ante_timeout_del_banco_destino_no_se_compensa_a_ciegas() {
    when(bancoExterno.acreditar(any())).thenThrow(new TimeoutException());
    when(bancoExterno.consultar(any())).thenReturn(Optional.empty());

    servicio.acreditar(solicitud);

    assertEquals(EN_REVISION, saga.estado());
    verify(cuentasClient, never()).reversar(any(), any());   // ← lo importante
}

@Test
void tras_cualquier_secuencia_el_sistema_cuadra() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que cuestan dinero
> - **Creer que `@Transactional` evita la condición de carrera.** Da atomicidad, no exclusión.
> - **Bloquear en el orden del request** en vez de un orden determinista → deadlock bajo carga.
> - **Compensar tras un timeout** sin consultar primero → dinero duplicado al revés.
> - **Validar contra el saldo contable** en vez del disponible.
> - **Permitir origen = destino:** genera un asiento que cuadra pero no tiene sentido.
> - **Reversar sin verificar fondos** en la cuenta destino.
> - **No modelar `EN_REVISION`:** el sistema se ve obligado a adivinar cuando no sabe.

## 📌 Notas Rápidas
- La pregunta *"¿cómo evitás que una transferencia se ejecute dos veces?"* tiene tres capas: idempotencia (reintentos), bloqueo ordenado (concurrencia) y consulta de estado (timeouts). Responder solo una es responder a medias.
- Que exista el estado `EN_REVISION` no es una debilidad del diseño: es reconocer que hay situaciones donde ningún algoritmo puede decidir correctamente. Los sistemas bancarios reales tienen equipos dedicados a esa cola.
- La ventana de dinero en tránsito es la razón por la que las transferencias interbancarias tradicionales tardan: no es lentitud técnica, es el ciclo de compensación entre entidades.
- Se implementa en [[Fase 2 - Transferencias e Idempotencia]] y se distribuye en [[Fase 6 - Microservicios y SAGA]].

## 🔗 Relaciones
- **Módulo**: [[Banca y Fintech]]
- **Anterior**: [[05 - Idempotencia Financiera]]
- **Siguiente**: [[07 - Medios de Pago y ISO 8583]]
- **Se conecta con**: [[06 - SAGA]], [[10 - Locks y Bloqueo Optimista]], [[05 - Transacciones]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

