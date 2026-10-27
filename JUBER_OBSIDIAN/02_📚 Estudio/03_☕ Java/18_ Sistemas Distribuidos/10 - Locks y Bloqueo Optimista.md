---
tags:
  - Java
  - Distribuido
  - Concurrencia
Creado: 2026-09-08
Relacionado:
  - Sistemas Distribuidos
  - 06 - Transferencias y Consistencia
Palabras clave:
  - bloqueo pesimista, optimista, version, lock distribuido, fencing token, deadlock, lease
---

## 🧠 Concepto

Cuando dos procesos quieren modificar el mismo dato a la vez, hay que decidir **quién gana**. Las opciones son tres, y elegir mal cuesta caro en los dos sentidos: o corrupción de datos, o un sistema que se arrastra.

```
Saldo: 1000        Proceso A              Proceso B
                   lee → 1000
                                          lee → 1000
                   valida 1000 ≥ 800 ✅
                                          valida 1000 ≥ 800 ✅
                   escribe 200
                                          escribe 200
Saldo: 200, pero se retiraron 1600 💥
```

Es la **actualización perdida** (*lost update*), y `@Transactional` por sí solo no la evita: el nivel de aislamiento por defecto (`READ COMMITTED`) la permite.

## 🔓 Bloqueo optimista

Supone que los conflictos son raros. No bloquea nada: **detecta** el conflicto al escribir.

```java
@Entity
public class CuentaEntity {
    @Id private UUID id;
    @Version private Long version;      // ← JPA lo maneja solo
    private BigDecimal saldo;
}
```

```sql
-- Lo que genera JPA al actualizar
UPDATE cuenta SET saldo = ?, version = 6
WHERE id = ? AND version = 5;           -- si otro ya subió a 6, afecta 0 filas → excepción
```

```java
@Retryable(retryFor = OptimisticLockingFailureException.class,
           maxAttempts = 3,
           backoff = @Backoff(delay = 50, multiplier = 2, random = true))
@Transactional
public void actualizarPerfil(UUID id, DatosPerfil datos) {
    var cuenta = repo.findById(id).orElseThrow();
    cuenta.actualizar(datos);
}
```

| Ventaja | Desventaja |
|---|---|
| No bloquea: máxima concurrencia | el trabajo perdido se repite |
| Sin deadlocks | mala experiencia si hay mucha contención |
| Funciona entre transacciones largas | requiere lógica de reintento |

> [!tip] Ideal cuando el conflicto es improbable
> Editar el perfil de un usuario, actualizar la descripción de un producto, cambiar una configuración. Si dos personas chocan una vez por semana, reintentar es más barato que bloquear siempre.

## 🔒 Bloqueo pesimista

Supone que el conflicto es probable. Bloquea la fila y hace esperar al resto.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT c FROM CuentaEntity c WHERE c.id = :id")
Optional<CuentaEntity> findByIdParaActualizar(@Param("id") UUID id);
```

```sql
SELECT * FROM cuenta WHERE id = ? FOR UPDATE;
-- la fila queda bloqueada hasta el COMMIT o ROLLBACK
```

| Modo | SQL | Permite |
|---|---|---|
| `PESSIMISTIC_READ` | `FOR SHARE` | otros leen, nadie escribe |
| `PESSIMISTIC_WRITE` | `FOR UPDATE` | nadie más toca la fila |
| — | `FOR UPDATE NOWAIT` | falla de inmediato si está tomada |
| — | `FOR UPDATE SKIP LOCKED` | ignora las filas bloqueadas |

```java
// Timeout: no esperar indefinidamente
@Lock(LockModeType.PESSIMISTIC_WRITE)
@QueryHints(@QueryHint(name = "jakarta.persistence.lock.timeout", value = "3000"))
Optional<CuentaEntity> findByIdParaActualizar(UUID id);
```

> [!danger] Un bloqueo sin timeout puede colgar el servicio
> Si una transacción toma un bloqueo y queda esperando algo lento (una llamada externa, por ejemplo), todas las que necesiten esa fila se apilan. Con suficiente tráfico, el pool de conexiones se agota y el servicio deja de responder — por una sola fila.

### Nunca llames a un servicio externo con un bloqueo tomado

```java
// ❌ el bloqueo dura lo que dure la llamada remota
@Transactional
public void procesar(UUID id) {
    var cuenta = repo.findByIdParaActualizar(id).orElseThrow();
    var resultado = servicioExterno.validar(cuenta);      // 💥 2 segundos con la fila tomada
    cuenta.aplicar(resultado);
}

// ✅ validar antes, bloquear solo para escribir
public void procesar(UUID id) {
    var resultado = servicioExterno.validar(datosDe(id));   // fuera de la transacción
    aplicarConBloqueo(id, resultado);                       // transacción corta
}
```

## 🔀 El deadlock y su eliminación

```
Transferencia A→B          Transferencia B→A
bloquea A ✅               bloquea B ✅
pide B ⏳                  pide A ⏳
              ☠️ ciclo de espera
```

Postgres detecta el ciclo y aborta una transacción, pero el usuario ve un error intermitente que solo aparece bajo carga.

```java
private ParDeCuentas bloquearEnOrden(CuentaId a, CuentaId b) {
    // SIEMPRE el identificador menor primero, sin importar quién es origen
    CuentaId primero = a.compareTo(b) < 0 ? a : b;
    CuentaId segundo = primero.equals(a) ? b : a;

    var c1 = repo.findByIdParaActualizar(primero).orElseThrow();
    var c2 = repo.findByIdParaActualizar(segundo).orElseThrow();
    return ParDeCuentas.de(c1, c2, a);
}
```

> [!tip] Ordenar los bloqueos hace el deadlock imposible, no improbable
> Si todas las transacciones piden los recursos en el mismo orden, no puede formarse un ciclo de espera. Es un resultado clásico de sistemas operativos, y una de las respuestas que más impresionan en una entrevista porque casi nadie la menciona.

## 🌐 Bloqueos distribuidos

Cuando el recurso no está en una base transaccional —un archivo, una API externa, un job que debe correr en una sola instancia— hace falta coordinación fuera de la base.

### El intento ingenuo

```java
public boolean adquirir(String recurso, String duenio, Duration ttl) {
    return redis.opsForValue()
        .setIfAbsent("lock:" + recurso, duenio, ttl);    // SET NX PX
}

public void liberar(String recurso, String duenio) {
    // El script Lua evita liberar el candado de OTRO
    redis.execute(SCRIPT_LIBERAR, List.of("lock:" + recurso), duenio);
}
```

```lua
-- Comparar y borrar, atómicamente
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end
```

> [!warning] Liberar sin verificar el dueño es un bug clásico
> Si tu candado expiró y otro proceso ya lo tomó, un `DEL` a ciegas libera **el candado del otro**. Hay que comparar el dueño y borrar en la misma operación atómica.

### El problema de fondo

> [!danger] Un bloqueo distribuido no garantiza exclusión mutua
> ```
> Proceso A adquiere el candado (TTL 10 s)
> Proceso A sufre una pausa de GC de 15 s
> El candado expira
> Proceso B adquiere el candado  ✅ legítimamente
> Proceso A despierta y sigue trabajando, creyendo que lo tiene 💥
> Los dos escriben al mismo tiempo
> ```
> Ninguna implementación basada en TTL resuelve esto, porque **el proceso pausado no puede saber que perdió el candado**. Es la crítica que Martin Kleppmann hizo al algoritmo Redlock en 2016, y sigue vigente.

### La solución: fencing tokens

El servicio de bloqueo entrega un número **monótonamente creciente**, y el recurso rechaza escrituras con un token viejo:

```java
public record Candado(String recurso, String duenio, long token) {}

// El recurso valida el token
@Transactional
public void escribir(UUID recursoId, Datos datos, long token) {
    long ultimoToken = repo.ultimoToken(recursoId);

    if (token <= ultimoToken) {
        throw new TokenObsoletoException(token, ultimoToken);   // ← A es rechazado
    }
    repo.escribir(recursoId, datos, token);
}
```

```
A obtiene token 33 → pausa de GC
B obtiene token 34 → escribe con 34 ✅
A despierta, escribe con 33 → RECHAZADO ✅
```

> [!tip] El fencing token traslada la garantía al recurso
> Es la única forma correcta de usar un bloqueo distribuido para corrección. Sin él, el candado sirve como **optimización** (evitar trabajo duplicado la mayoría de las veces) pero **no como garantía**.

### Cuándo alcanza y cuándo no

| Uso | ¿Basta un lock con TTL? |
|---|---|
| Evitar que dos instancias corran el mismo job | ✅ el peor caso es trabajo duplicado |
| Reducir contención sobre un recurso caro | ✅ es una optimización |
| Garantizar que solo uno escriba dinero | ❌ necesitás fencing o una transacción de base |

> [!warning] Si el recurso está en una base transaccional, no uses un lock distribuido
> `SELECT FOR UPDATE` da exclusión mutua **real**, sin TTL, sin pausas de GC que la invaliden y sin infraestructura extra. Los bloqueos distribuidos son para recursos que no tienen transacciones propias.

### Alternativa: la elección de líder

```java
// Con ShedLock: un solo nodo ejecuta el job programado
@Scheduled(cron = "0 0 3 * * *")
@SchedulerLock(name = "cierreDiario",
               lockAtMostFor = "PT2H",      // por si el nodo muere
               lockAtLeastFor = "PT1M")     // evita doble ejecución por desfase de relojes
public void cierreDiario() { ... }
```

Para elección de líder con garantías fuertes, lo correcto es un sistema de consenso —etcd, ZooKeeper, Consul— que resuelve el problema con quórum en vez de con TTL. Ver [[13 - Consenso y Raft]].

## 🎯 Cómo elegir

```
¿El recurso está en una base transaccional?
├── SÍ → ¿hay mucha contención sobre la misma fila?
│        ├── SÍ → bloqueo pesimista (FOR UPDATE), en orden determinista
│        └── NO → bloqueo optimista (@Version) con reintento
└── NO → ¿la corrección depende de la exclusión?
         ├── SÍ → fencing tokens, o consenso
         └── NO → lock con TTL alcanza (es una optimización)
```

| Caso | Mecanismo |
|---|---|
| Transferencia entre cuentas | pesimista + orden determinista |
| Editar un perfil | optimista |
| Reservar el último ítem de stock | pesimista |
| Consumir un límite diario | pesimista sobre la fila de consumo |
| Job nocturno en un solo nodo | ShedLock (TTL alcanza) |
| Escribir un archivo compartido | fencing token |

## 🧪 Tests

```java
@Test
void dos_escrituras_concurrentes_no_pierden_ninguna_actualizacion() {
    ejecutarEnParalelo(2, () -> servicio.incrementar(id));
    assertEquals(2, repo.findById(id).get().contador());
}

@Test
void el_bloqueo_optimista_detecta_el_conflicto() {
    var a = repo.findById(id).orElseThrow();
    var b = repo.findById(id).orElseThrow();
    a.actualizar(datos1); repo.saveAndFlush(a);
    b.actualizar(datos2);
    assertThrows(OptimisticLockingFailureException.class, () -> repo.saveAndFlush(b));
}

@Test
void transferencias_cruzadas_simultaneas_no_producen_deadlock() {
    assertTimeoutPreemptively(Duration.ofSeconds(30), () ->
        ejecutarEnParalelo(100, alternando(
            () -> transferir(a, b, "10"),
            () -> transferir(b, a, "10"))));
}

@Test
void un_token_obsoleto_es_rechazado() {
    servicio.escribir(recurso, datos, 34L);
    assertThrows(TokenObsoletoException.class, () -> servicio.escribir(recurso, datos, 33L));
}

@Test
void el_bloqueo_respeta_el_timeout_configurado() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que producen corrupción o caídas
> - **Creer que `@Transactional` evita la actualización perdida**: da atomicidad, no exclusión.
> - **Bloquear en el orden del request**: receta del deadlock.
> - **Bloqueos sin timeout**: agotan el pool de conexiones.
> - **Llamar a un servicio externo con un bloqueo tomado.**
> - **Liberar un lock distribuido sin verificar el dueño.**
> - **Usar un lock con TTL para corrección** sin fencing tokens.
> - **Lock distribuido sobre un recurso que ya está en una base transaccional.**
> - **Bloqueo optimista sin lógica de reintento**: el error llega al usuario.
> - **Reintentar sin jitter**: los conflictos se sincronizan y vuelven a chocar.

## 📌 Notas Rápidas
- El escenario "pausa de GC mientras tengo el candado" parece rebuscado y no lo es: una pausa larga de GC, una VM suspendida o un contenedor con CPU limitada producen exactamente eso. Todo lo que se detiene por más tiempo que el TTL rompe la exclusión.
- El artículo de Kleppmann *"How to do distributed locking"* (2016) y la respuesta de antirez son la mejor lectura sobre el tema, y la discusión sigue siendo relevante.
- La regla práctica: **si podés resolverlo con la base, resolvelo con la base**. Los bloqueos distribuidos agregan un modo de fallo que la mayoría de los sistemas no necesita.
- Se implementa en [[Fase 2 - Transferencias e Idempotencia]].

## 🔗 Relaciones
- **Módulo**: [[Sistemas Distribuidos]]
- **Anterior**: [[09 - CQRS Distribuido]]
- **Siguiente**: [[11 - Relojes y Causalidad]]
- **Se conecta con**: [[06 - Transferencias y Consistencia]], [[05 - Transacciones]], [[13 - Consenso y Raft]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

