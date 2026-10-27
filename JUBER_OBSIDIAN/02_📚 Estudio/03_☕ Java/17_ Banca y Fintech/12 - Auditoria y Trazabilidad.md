---
tags:
  - Java
  - Banca
  - Seguridad
Creado: 2026-09-08
Relacionado:
  - Banca y Fintech
  - 03 - Ledger y Partida Doble
Palabras clave:
  - auditoria, append-only, no repudio, cadena de hashes, WORM, retencion, trazabilidad
---

## 🧠 Concepto

La **auditoría** responde a una pregunta que un regulador puede hacer en cualquier momento:

> ¿Quién hizo qué, cuándo, desde dónde, y cómo sabemos que ese registro no fue alterado?

No es logging. Los logs son para diagnosticar; la auditoría es **evidencia** — tiene requisitos legales de integridad y retención, y en un juicio o una inspección se presenta como prueba.

| | Logs | Auditoría |
|---|---|---|
| Propósito | diagnosticar | demostrar |
| Audiencia | el equipo | regulador, auditor, juez |
| Retención | días o semanas | años, por normativa |
| ¿Se pueden borrar? | sí, rotan | ❌ nunca |
| ¿Se pueden perder? | tolerable | ❌ inaceptable |
| Formato | libre | estructurado y estable |

## 🎯 Los cuatro atributos

| Atributo | Qué significa | Cómo se logra |
|---|---|---|
| **Integridad** | nadie lo alteró | append-only + cadena de hashes |
| **Completitud** | no falta nada | registro sistemático, no manual |
| **Atribución** | se sabe quién fue | identidad en cada evento |
| **No repudio** | el actor no puede negarlo | firma o evidencia criptográfica |

## 🗄️ El registro

```sql
CREATE TABLE auditoria (
    id             BIGSERIAL PRIMARY KEY,
    ocurrido_en    TIMESTAMPTZ  NOT NULL DEFAULT now(),

    -- Quién
    usuario_id     UUID,
    username       VARCHAR(60),        -- desnormalizado a propósito
    roles          VARCHAR(200),
    ip             INET,
    user_agent     TEXT,

    -- Qué
    accion         VARCHAR(60)  NOT NULL,
    recurso_tipo   VARCHAR(40),
    recurso_id     VARCHAR(60),
    resultado      VARCHAR(20)  NOT NULL,   -- OK | RECHAZADO | ERROR

    -- Contexto
    trace_id       VARCHAR(40),
    canal          VARCHAR(20),             -- WEB | MOVIL | API | BATCH | SUCURSAL
    detalle        JSONB,

    -- Integridad
    hash_previo    VARCHAR(64),
    hash           VARCHAR(64)  NOT NULL
);

CREATE INDEX idx_auditoria_fecha   ON auditoria(ocurrido_en DESC);
CREATE INDEX idx_auditoria_usuario ON auditoria(usuario_id, ocurrido_en DESC);
CREATE INDEX idx_auditoria_recurso ON auditoria(recurso_tipo, recurso_id);
CREATE INDEX idx_auditoria_trace   ON auditoria(trace_id);
```

> [!tip] `username` y `roles` se desnormalizan a propósito
> Si mañana el usuario se renombra, cambia de rol o se da de baja, la auditoría tiene que seguir diciendo **quién era en ese momento**. Es una foto histórica, no una vista viva. Un `JOIN` contra la tabla de usuarios daría la respuesta de hoy, no la de entonces.

### Inmutabilidad impuesta por la base

```sql
REVOKE UPDATE, DELETE ON auditoria FROM corebank_app;

CREATE RULE auditoria_sin_update AS ON UPDATE TO auditoria DO INSTEAD NOTHING;
CREATE RULE auditoria_sin_delete AS ON DELETE TO auditoria DO INSTEAD NOTHING;
```

> [!danger] La inmutabilidad no se confía al código
> "Nuestro código nunca hace `UPDATE` sobre esa tabla" no es un control: es una promesa. Un bug, una migración o una consulta manual la rompen. El permiso revocado a nivel de base es un control real, verificable por un auditor.

## ⛓️ Cadena de hashes

Revocar permisos protege contra la aplicación, pero no contra alguien con acceso administrativo a la base. Encadenar los registros hace **detectable** cualquier alteración:

```java
public void registrar(EventoAuditoria evento) {
    String hashPrevio = repo.ultimoHash().orElse(GENESIS);

    String contenido = String.join("|",
        evento.ocurridoEn().toString(),
        String.valueOf(evento.usuarioId()),
        evento.accion(),
        String.valueOf(evento.recursoId()),
        evento.resultado().name(),
        evento.detalleSerializado(),
        hashPrevio);                          // ← el eslabón

    String hash = sha256Hex(contenido);
    repo.insertar(evento.conHash(hashPrevio, hash));
}
```

```
registro 1 · hash = H(datos₁ + GENESIS)
registro 2 · hash = H(datos₂ + hash₁)
registro 3 · hash = H(datos₃ + hash₂)
             ▲
             └─ alterar el registro 2 invalida el 3, el 4 y todos los siguientes
```

```java
public ResultadoVerificacion verificarCadena(Instant desde, Instant hasta) {
    String esperado = repo.hashAnteriorA(desde);

    for (RegistroAuditoria r : repo.entre(desde, hasta)) {
        if (!r.hashPrevio().equals(esperado)) {
            return ResultadoVerificacion.rota(r.id(), "eslabón no coincide");
        }
        String recalculado = sha256Hex(r.contenidoCanonico() + r.hashPrevio());
        if (!recalculado.equals(r.hash())) {
            return ResultadoVerificacion.rota(r.id(), "contenido alterado");
        }
        esperado = r.hash();
    }
    return ResultadoVerificacion.intacta();
}
```

> [!warning] La cadena detecta, no impide
> Alguien con acceso total podría recalcular toda la cadena desde el punto alterado. Para cerrar esa puerta se publica periódicamente el último hash en un lugar fuera de control del administrador: otro sistema, almacenamiento WORM, o un correo firmado a los auditores. Si el hash publicado no coincide con el recalculado, la manipulación queda expuesta.

> [!tip] La verificación tiene que ser un job programado
> Una cadena que nadie verifica no sirve de nada. Se corre en cada cierre y se alerta si se rompe.

## 📋 Qué se audita

| Acción | ¿Auditar? |
|---|---|
| Login exitoso y fallido | ✅ |
| Bloqueo de usuario por intentos | ✅ |
| Cambio de contraseña o de MFA | ✅ |
| Cualquier operación con dinero | ✅ |
| Reversa (con motivo y operador) | ✅ |
| Acceso denegado a un recurso ajeno | ✅ |
| Alta, baja o cambio de rol | ✅ |
| Consulta de datos de otro cliente por staff | ✅ |
| Exportación masiva de datos | ✅ |
| Cambio de configuración o de límites | ✅ |
| Ejecución de procesos batch | ✅ |
| Consulta del propio saldo | ❌ ruido |
| Health checks | ❌ |

> [!tip] Auditá los rechazos con la misma prioridad que los éxitos
> Diez accesos denegados seguidos sobre cuentas ajenas es la señal más clara de un ataque en curso. Si solo registrás lo que salió bien, ese patrón es invisible.

### Registro automático

```java
@Aspect
@Component
public class AspectoAuditoria {

    @AfterReturning(pointcut = "@annotation(auditable)", returning = "resultado")
    public void exito(JoinPoint jp, Auditable auditable, Object resultado) {
        auditoria.registrar(construir(jp, auditable, OK, null));
    }

    @AfterThrowing(pointcut = "@annotation(auditable)", throwing = "ex")
    public void fallo(JoinPoint jp, Auditable auditable, Throwable ex) {
        var resultado = ex instanceof AccessDeniedException ? RECHAZADO : ERROR;
        auditoria.registrar(construir(jp, auditable, resultado, ex));
    }
}
```

```java
@Auditable(accion = "TRANSFERENCIA", recurso = "CUENTA")
public ResultadoTransferencia transferir(ComandoTransferir cmd) { ... }
```

> [!warning] Auditar a mano dentro de cada servicio no funciona
> Se olvida en la mitad de los métodos y nadie lo nota hasta la inspección. Con aspecto, interceptor o filtro, el registro es sistemático: si el método tiene la anotación, se audita.

## 🔗 Auditoría, ledger y logs

Tres registros distintos, con propósitos distintos:

| Registro | Qué guarda | Ejemplo |
|---|---|---|
| **Ledger** | el efecto contable | "-500 en ACC-001, +500 en ACC-002" |
| **Auditoría** | la acción y su actor | "juber transfirió 500 desde la app a las 14:32 desde IP x" |
| **Logs** | el detalle técnico | "consulta de saldo tardó 43ms, 2 reintentos" |

Los tres se cruzan por `trace_id`, y esa es la propiedad más útil del conjunto: desde un asiento podés llegar a quién lo originó, y desde ahí a qué hizo el sistema por dentro.

## 📅 Retención

| Tipo de dato | Retención típica |
|---|---|
| Movimientos y asientos | 10 años |
| Auditoría de operaciones | 5-10 años |
| Auditoría de accesos | 1-2 años |
| Logs técnicos | 30-90 días |

Los plazos concretos los fija la normativa de cada país y sector. Lo importante para el diseño: **la auditoría vive mucho más que los logs**, así que no puede estar en el mismo lugar.

```java
@Scheduled(cron = "0 0 4 1 * *")   // mensual
public void archivar() {
    Instant corte = Instant.now().minus(Duration.ofDays(365));

    // Se mueve a almacenamiento WORM, no se borra
    var lote = repo.anterioresA(corte);
    almacenWorm.escribir(lote);              // S3 Object Lock / Azure immutable blob
    repo.marcarArchivado(lote);
}
```

> [!tip] WORM: Write Once, Read Many
> Almacenamiento que impide modificar o borrar durante un período configurado, **ni siquiera con permisos de administrador**. S3 Object Lock en modo *compliance* y los blobs inmutables de Azure lo implementan. Es lo que convierte "prometemos no borrarlo" en un control técnico.

## ⚖️ Auditoría vs protección de datos

Hay una tensión real: la normativa de protección de datos reconoce el derecho de supresión, y la financiera obliga a conservar.

> [!warning] La obligación legal de conservar suele prevalecer
> Un registro de auditoría de una transacción financiera normalmente **no se borra** ante un pedido de supresión, porque existe una obligación legal que lo ampara. Lo que sí se hace es minimizar: guardar identificadores en vez de datos personales, y limitar quién puede consultarlos.
> Es una decisión con criterio legal, no técnica. **No la tomes vos solo** — pero sabé que existe y planteala.

```java
// ✅ referencia mínima, sin datos personales duplicados
detalle = Map.of("cuentaId", cuenta.id(), "monto", monto);

// ❌ innecesario: duplica datos personales en un registro de retención larga
detalle = Map.of("nombre", cliente.nombre(), "email", cliente.email(),
                 "documento", cliente.documento());
```

## 🧪 Tests

```java
@Test
void la_tabla_de_auditoria_rechaza_update_y_delete() {
    Long id = auditoria.registrar(evento());
    assertEquals(0, jdbc.update("UPDATE auditoria SET accion='X' WHERE id=?", id));
    assertEquals(0, jdbc.update("DELETE FROM auditoria WHERE id=?", id));
    assertTrue(repo.existsById(id));
}

@Test
void alterar_un_registro_rompe_la_cadena() {
    IntStream.range(0, 10).forEach(i -> auditoria.registrar(evento()));
    jdbc.execute("UPDATE auditoria SET detalle='{\"x\":1}' WHERE id=5");  // fuerza bruta
    assertFalse(verificador.verificarCadena(desde, hasta).intacta());
}

@Test
void un_acceso_denegado_se_audita() {
    assertThrows(AccessDeniedException.class, () -> api.verCuentaAjena());
    assertTrue(repo.existePorAccionYResultado("CONSULTA_CUENTA", RECHAZADO));
}

@Test
void el_registro_conserva_el_username_aunque_el_usuario_se_elimine() { ... }

@Test
void ningun_registro_de_auditoria_contiene_datos_sensibles() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que se caen en una inspección
> - **Confiar la inmutabilidad al código** en vez de revocar permisos.
> - **`JOIN` contra usuarios** en vez de desnormalizar: devuelve el estado de hoy, no el de entonces.
> - **Auditar solo los éxitos**: los ataques se ven en los rechazos.
> - **Auditar a mano** en cada servicio: se olvida y nadie lo nota.
> - **Mezclar auditoría con logs técnicos**: se rotan y se pierde la evidencia.
> - **Cadena de hashes que nadie verifica**: decorativa.
> - **Guardar datos personales de más** en un registro de retención larga.
> - **Auditoría en la misma base y sin respaldo**: si la comprometen, se comprometen las dos cosas.

## 📌 Notas Rápidas
- La pregunta *"¿por qué la auditoría no debe poder editarse ni siquiera por un admin?"* es una buena forma de detectar si alguien entendió el concepto: porque el propio admin es uno de los actores que la auditoría tiene que poder incriminar.
- La cadena de hashes es la misma idea que una blockchain, sin la parte distribuida. No hace falta una blockchain para tener un registro verificable: hace falta encadenar y publicar el hash fuera de tu control.
- El `trace_id` compartido entre ledger, auditoría y logs es lo que convierte tres registros sueltos en una historia reconstruible. Vale la pena diseñarlo desde el principio.
- Se implementa en [[Fase 3 - Seguridad y Auditoria]] y se apoya en la observabilidad de [[Fase 7 - Observabilidad y Despliegue]].

## 🔗 Relaciones
- **Módulo**: [[Banca y Fintech]]
- **Anterior**: [[11 - PCI-DSS y HSM]]
- **Siguiente**: [[13 - Antifraude y Limites]]
- **Se conecta con**: [[03 - Ledger y Partida Doble]], [[06 - Logs Observabilidad y Monitoreo]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

