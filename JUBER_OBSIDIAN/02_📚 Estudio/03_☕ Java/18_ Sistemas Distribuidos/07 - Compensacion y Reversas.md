---
tags:
  - Java
  - Distribuido
  - Patrones
Creado: 2026-09-08
Relacionado:
  - Sistemas Distribuidos
  - 06 - SAGA
Palabras clave:
  - compensacion, rollback, pivote, retriable, compensable, irreversible, semantica
---

## 🧠 Concepto

Un **rollback** deshace: la base restaura el estado anterior y no queda rastro de que algo pasó.

Una **compensación** no deshace: aplica una operación nueva que produce el efecto contrario, y **los dos hechos quedan registrados**.

```
ROLLBACK                        COMPENSACIÓN
saldo: 1000                     saldo: 1000
  ↓ débito 200                    ↓ débito 200
saldo: 800                      saldo: 800  · asiento #1
  ↓ rollback                      ↓ compensación
saldo: 1000                     saldo: 1000 · asiento #2
(nunca pasó nada)               (pasaron dos cosas)
```

> [!tip] En un sistema financiero, la compensación es lo correcto — no un mal menor
> El cliente tiene derecho a ver que hubo un débito y que se revirtió. El auditor también. Un rollback que borra la evidencia sería inaceptable aunque fuera técnicamente posible.

## 🔀 Los tres tipos de paso

En una SAGA, no todos los pasos son iguales. La distinción es fundamental para diseñarla bien:

| Tipo | Qué significa | Ejemplo |
|---|---|---|
| **Compensable** | se puede deshacer con una operación inversa | reservar fondos, debitar |
| **Pivote** | el punto de no retorno | el débito confirmado ante la cámara |
| **Retriable** | después del pivote: solo puede avanzar | acreditar, notificar, confirmar |

```
[ compensable ][ compensable ][ PIVOTE ][ retriable ][ retriable ]
       ↑              ↑            │           ↓            ↓
       └──── se compensan ─────────┘      solo se reintentan hasta lograrlo
```

> [!danger] Después del pivote no se puede volver atrás
> Un paso retriable **tiene que terminar**: se reintenta indefinidamente (con tope y escalado). Si un paso posterior al pivote fallara de forma definitiva, el sistema queda en un estado que ninguna compensación puede arreglar.
>
> Por eso el diseño correcto pone todo lo que puede fallar **antes** del pivote, y todo lo irreversible **después**.

```java
public enum TipoPaso { COMPENSABLE, PIVOTE, RETRIABLE }

// Transferencia interbancaria
reservarFondos      → COMPENSABLE   (se libera la reserva)
validarBeneficiario → COMPENSABLE   (no tiene efecto)
debitarOrigen       → COMPENSABLE   (asiento de reversa)
enviarACamara       → PIVOTE        ← desde acá no hay vuelta atrás
registrarEnvio      → RETRIABLE
notificarCliente    → RETRIABLE
```

## 🧩 Diseñar una compensación

### Debe ser idempotente

```java
@Transactional
public void compensarDebito(UUID asientoId, String motivo) {
    if (libro.tieneReversa(asientoId)) {
        log.debug("asiento {} ya compensado", asientoId);
        return;                          // ← no es error: ya está hecho
    }
    libro.registrar(reversaDe(asientoId, motivo));
}
```

Una compensación puede ejecutarse dos veces —por reintento del orquestador, o porque el recuperador la reanudó—. Si no es idempotente, compensa el doble.

### Debe poder fallar de forma controlada

```java
public ResultadoCompensacion compensar(SagaTransferencia saga) {
    try {
        cuentas.reversar(saga.asientoDebitoId(), motivo(saga));
        return ResultadoCompensacion.exitosa();

    } catch (SaldoInsuficienteException e) {
        // El destinatario ya usó el dinero: no hay compensación técnica posible
        saga.pasarAEnRevision("compensación imposible: " + e.getMessage());
        alertas.escalar(saga);
        return ResultadoCompensacion.requiereIntervencion(e.getMessage());
    }
}
```

> [!danger] Una compensación puede ser imposible
> Si transferiste S/ 5.000 y el destinatario ya gastó S/ 4.000, revertir dejaría su cuenta en descubierto. El sistema **no debe forzarlo**: lo correcto es marcar la saga en revisión y escalar.
> Forzar la compensación convierte un problema operativo en uno legal.

### Debe registrar el vínculo

```sql
ALTER TABLE asiento ADD COLUMN compensa_a UUID REFERENCES asiento(id);
CREATE UNIQUE INDEX idx_una_compensacion_por_asiento
    ON asiento(compensa_a) WHERE compensa_a IS NOT NULL;
```

El índice único garantiza a nivel de base que un asiento se compensa **una sola vez**, sin depender de que el código lo verifique.

## 🚫 Operaciones no compensables

| Operación | ¿Compensable? | Qué se puede hacer |
|---|---|---|
| Débito contable | ✅ | asiento inverso |
| Reserva de fondos | ✅ | liberar |
| Reserva de stock | ✅ | liberar |
| Correo enviado | ❌ | mandar otro corrigiendo |
| SMS enviado | ❌ | nada |
| Documento impreso | ❌ | anular manualmente |
| Mercadería entregada | ❌ | proceso de devolución |
| Pago irrevocable a un tercero | ❌ | reclamo |

> [!warning] Lo irreversible va al final, siempre
> Es la regla de diseño más importante de esta nota. Si un paso no se puede deshacer, tiene que ocurrir cuando ya no queda nada que pueda fallar después.
>
> ```
> ❌ notificar → debitar → acreditar     (si acreditar falla, ya notificaste)
> ✅ debitar → acreditar → notificar     (notificás cuando todo salió bien)
> ```

## ⚖️ Compensación parcial

Cuando la SAGA falla en el paso 4 de 5, hay que compensar 3, 2 y 1 — **en ese orden**, y solo los que se aplicaron:

```java
private void compensar(SagaTransferencia saga) {
    var pasosAplicados = saga.pasosAplicados();

    // Orden inverso, y solo lo que efectivamente se ejecutó
    for (PasoAplicado paso : pasosAplicados.reversed()) {
        if (paso.tipo() != COMPENSABLE) continue;

        try {
            compensadores.get(paso.nombre()).compensar(saga, paso);
            saga.marcarCompensado(paso.nombre());
            repo.save(saga);                    // ← persistir tras CADA compensación

        } catch (Exception e) {
            saga.pasarAEnRevision("falló compensar " + paso.nombre());
            alertas.escalar(saga);
            return;                             // no seguir a ciegas
        }
    }
    saga.fallida();
    repo.save(saga);
}
```

> [!tip] Persistir después de cada compensación individual
> Si el proceso muere en medio de la compensación, al reiniciar tiene que saber **cuáles ya se compensaron**. Sin ese registro, el recuperador vuelve a compensar todo — y sin idempotencia, duplica al revés.

## ❓ Compensar ante un timeout

El caso más delicado: llamaste al servicio externo y no respondió. **No sabés si se ejecutó.**

```java
private void manejarTimeout(SagaTransferencia saga) {
    // ❌ NO compensar: si se ejecutó, la compensación duplica el efecto inverso
    // ❌ NO reintentar a ciegas: si se ejecutó, duplica hacia adelante
    // ✅ CONSULTAR el estado real

    Optional<EstadoOperacion> real = bancoExterno.consultar(saga.id().toString());

    if (real.isEmpty()) {
        saga.pasarAEnRevision("estado desconocido tras timeout");
        alertas.escalar(saga);
        return;
    }

    switch (real.get()) {
        case APLICADA  -> { saga.acreditada(real.get().referencia()); avanzar(saga); }
        case RECHAZADA -> compensar(saga);
        case NO_EXISTE -> reintentar(saga);          // nunca llegó
    }
}
```

> [!danger] Toda operación externa necesita ser consultable
> Si el proveedor no expone un endpoint de consulta por referencia, no tenés forma de resolver un timeout. Es un requisito que hay que exigir **al diseñar la integración**, no cuando ya está en producción.

## 🔍 Compensación semántica vs técnica

| | Técnica | Semántica |
|---|---|---|
| Qué hace | invierte los valores | aplica la operación de negocio inversa |
| Ejemplo | `saldo += 200` | registrar un asiento de reversa |
| Auditable | no deja rastro | deja los dos hechos |
| Efectos secundarios | los ignora | los considera (comisiones, límites, notificaciones) |

```java
// ❌ Técnica: revierte el número, ignora todo lo demás
cuenta.setSaldo(cuenta.getSaldo().add(monto));

// ✅ Semántica: es una operación de negocio con todas sus consecuencias
public void reversar(UUID asientoId, String motivo, String operador) {
    Asiento reversa = libro.registrar(asientoInversoDe(asientoId, motivo, operador));
    limites.liberarConsumo(asientoId);                    // devuelve el cupo usado
    eventos.publicar(new TransferenciaReversada(asientoId, reversa.id(), motivo));
    auditoria.registrar(REVERSA, asientoId, operador);
}
```

> [!warning] Una compensación técnica olvida los efectos laterales
> El límite diario consumido, la notificación ya enviada, la métrica contada, la regla antifraude disparada. La compensación semántica los contempla porque **es** una operación del dominio, no un ajuste de números.

## 🧪 Tests

```java
@Test
void tras_compensar_el_saldo_vuelve_al_valor_original() {
    var antes = saldos.contable(cuenta, PEN);
    var saga = ejecutarSagaQueFalla();
    assertEquals(antes, saldos.contable(cuenta, PEN));
}

@Test
void la_compensacion_deja_los_dos_asientos() {
    ejecutarSagaQueFalla();
    assertEquals(2, libro.contarAsientosDe(cuenta));     // original + reversa
}

@Test
void compensar_dos_veces_produce_un_solo_efecto() {
    compensador.compensar(saga);
    compensador.compensar(saga);
    assertEquals(1, libro.contarReversasDe(asientoId));
}

@Test
void si_el_destino_ya_gasto_el_dinero_la_saga_queda_en_revision() {
    gastarTodoEn(destino);
    compensador.compensar(saga);
    assertEquals(EN_REVISION, saga.estado());
    verify(alertas).escalar(saga);
}

@Test
void ante_timeout_se_consulta_el_estado_antes_de_compensar() {
    when(externo.acreditar(any())).thenThrow(new TimeoutException());
    when(externo.consultar(any())).thenReturn(Optional.of(APLICADA));

    orquestador.avanzar(sagaId);

    verify(cuentas, never()).reversar(any(), any());     // ← no compensó
    assertEquals(COMPLETADA, saga.estado());
}

@Test
void la_compensacion_parcial_deshace_solo_los_pasos_aplicados() { ... }

@Test
void el_consumo_del_limite_se_libera_al_compensar() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que dejan dinero mal
> - **Compensar tras timeout sin consultar**: duplica el efecto inverso.
> - **Compensación no idempotente**: el reintento compensa dos veces.
> - **Compensar en orden de ejecución** en vez de inverso.
> - **No persistir tras cada compensación individual.**
> - **Forzar una compensación imposible** dejando la cuenta en descubierto.
> - **Compensación técnica**: olvida límites, notificaciones y auditoría.
> - **Pasos irreversibles antes del pivote**: no hay forma de arreglarlo.
> - **No distinguir compensable de retriable**: se intenta compensar lo que solo se puede reintentar.
> - **No probar el camino de compensación**: el que falla en producción.

## 📌 Notas Rápidas
- La distinción compensable / pivote / retriable es el aporte más práctico de esta nota: obliga a ordenar los pasos con criterio, y ese orden es el 80% de una SAGA bien diseñada.
- Que exista `EN_REVISION` y una cola de intervención manual no es una debilidad del diseño: los sistemas bancarios reales tienen equipos dedicados a esa cola. Pretender que todo se resuelve solo es lo que produce inconsistencias silenciosas.
- Antes de integrar con un tercero, verificá que exponga consulta de estado por referencia. Sin eso, cada timeout es una intervención manual.
- Se implementa en [[Fase 6 - Microservicios y SAGA]].

## 🔗 Relaciones
- **Módulo**: [[Sistemas Distribuidos]]
- **Anterior**: [[06 - SAGA]]
- **Siguiente**: [[08 - Event Sourcing]]
- **Se conecta con**: [[06 - Transferencias y Consistencia]], [[03 - Ledger y Partida Doble]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

