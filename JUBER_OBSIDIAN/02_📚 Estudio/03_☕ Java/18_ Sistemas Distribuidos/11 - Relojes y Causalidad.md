---
tags:
  - Java
  - Distribuido
  - Fundamentos
Creado: 2026-09-08
Relacionado:
  - Sistemas Distribuidos
  - 02 - CAP PACELC y Consistencia
Palabras clave:
  - reloj, NTP, deriva, monotonico, Lamport, vector clock, HLC, causalidad, UUIDv7
---

## 🧠 Concepto

En una sola máquina, "qué pasó primero" es una pregunta trivial: hay un reloj y todos lo miran.

En un sistema distribuido **no existe un ahora compartido**. Cada nodo tiene su propio reloj, todos derivan, y ordenar eventos por timestamp produce respuestas incorrectas.

```
Nodo A (reloj +50 ms)     escribe X en t=1000  (real: 950)
Nodo B (reloj -30 ms)     escribe Y en t=980   (real: 1010)

Ordenado por timestamp: Y antes que X
Orden real:             X antes que Y  ❌
```

## ⏰ Relojes físicos

### Los dos relojes de la JVM

| | `System.currentTimeMillis()` | `System.nanoTime()` |
|---|---|---|
| Qué mide | tiempo del calendario | contador monótono |
| Salta hacia atrás | ✅ sí (NTP, ajuste manual) | ❌ nunca |
| Comparable entre máquinas | ✅ (aproximadamente) | ❌ sin significado |
| Para | fechas y timestamps | **medir duraciones** |

```java
// ❌ El resultado puede ser NEGATIVO si NTP ajusta el reloj en el medio
long inicio = System.currentTimeMillis();
operacion();
long duracion = System.currentTimeMillis() - inicio;

// ✅ Monótono: nunca retrocede
long inicio = System.nanoTime();
operacion();
Duration duracion = Duration.ofNanos(System.nanoTime() - inicio);
```

> [!danger] Medir duraciones con el reloj de pared es un bug latente
> Funciona el 99,99% del tiempo y falla justo cuando NTP hace un ajuste. Los síntomas son absurdos: latencias negativas, timeouts que expiran de inmediato, reintentos que no esperan. Y no se reproduce.

```java
// Micrometer usa el reloj correcto internamente
Timer.Sample muestra = Timer.start(registry);
operacion();
muestra.stop(registry.timer("operacion.duracion"));
```

### La deriva

| Fuente | Deriva típica |
|---|---|
| Cristal de cuarzo sin corregir | ~10 s por día |
| Con NTP en internet | 1-50 ms |
| Con NTP en la misma red | < 1 ms |
| Máquina virtual sobrecargada | **cientos de ms**, impredecible |

> [!warning] En una VM el reloj puede saltar
> Cuando el hipervisor suspende una máquina virtual y la reanuda, el reloj puede saltar hacia adelante de golpe. En contenedores con CPU limitada, pasa lo mismo por otras razones. No es una hipótesis teórica.

## 🚫 Lo que no se puede hacer con timestamps

```java
// ❌ Ordenar eventos de nodos distintos
eventos.sort(comparing(Evento::timestamp));       // el orden puede ser incorrecto

// ❌ Decidir quién ganó un conflicto (last-write-wins)
if (nuevo.timestamp() > actual.timestamp()) { sobrescribir(); }
```

> [!danger] "Last write wins" pierde escrituras en silencio
> Si el reloj del nodo A está 100 ms adelantado, sus escrituras "ganan" siempre — aunque hayan ocurrido antes. Las escrituras del nodo B se descartan sin error, sin log y sin que nadie se entere.
> Es una de las causas más difíciles de diagnosticar de pérdida de datos en sistemas replicados.

```java
// ❌ Expirar por comparación de relojes entre nodos
if (Instant.now().isAfter(token.expiraEn())) { rechazar(); }
// si los relojes difieren, un token válido se rechaza o uno vencido se acepta

// ✅ Tolerancia explícita
private static final Duration TOLERANCIA = Duration.ofSeconds(30);
if (Instant.now().minus(TOLERANCIA).isAfter(token.expiraEn())) { rechazar(); }
```

> [!tip] Los JWT llevan tolerancia por esta razón
> El parámetro *clock skew* de las bibliotecas de JWT (típicamente 30-60 s) existe porque los relojes del emisor y el validador no coinciden. Ponerlo en cero produce rechazos aleatorios.

## 🔢 Relojes lógicos

Si el tiempo físico no sirve para ordenar, se usa el **orden causal**: qué evento pudo haber influido en cuál.

### Relojes de Lamport

```java
public class RelojLamport {
    private final AtomicLong contador = new AtomicLong();

    /** Evento local */
    public long tick() { return contador.incrementAndGet(); }

    /** Al recibir un mensaje: sincronizar con el emisor */
    public long alRecibir(long relojDelEmisor) {
        return contador.updateAndGet(actual -> Math.max(actual, relojDelEmisor) + 1);
    }
}
```

Garantía: **si A causó B, entonces L(A) < L(B)**.

> [!warning] La implicación no vale al revés
> `L(A) < L(B)` **no** significa que A causó B: pueden ser eventos concurrentes sin relación. Los relojes de Lamport dan un orden total, pero no distinguen "ocurrió antes" de "son independientes".

### Relojes vectoriales

Resuelven esa limitación: cada nodo lleva un contador por nodo.

```java
public record RelojVectorial(Map<String, Long> contadores) {

    public RelojVectorial tick(String nodo) {
        var nuevo = new HashMap<>(contadores);
        nuevo.merge(nodo, 1L, Long::sum);
        return new RelojVectorial(nuevo);
    }

    public Relacion comparar(RelojVectorial otro) {
        boolean algunoMayor = false, algunoMenor = false;

        for (String nodo : union(contadores.keySet(), otro.contadores.keySet())) {
            long a = contadores.getOrDefault(nodo, 0L);
            long b = otro.contadores.getOrDefault(nodo, 0L);
            if (a > b) algunoMayor = true;
            if (a < b) algunoMenor = true;
        }

        if (algunoMayor && algunoMenor) return CONCURRENTES;   // ← conflicto real
        if (algunoMayor)                return POSTERIOR;
        if (algunoMenor)                return ANTERIOR;
        return IGUALES;
    }
}
```

```
A = {n1:2, n2:1}    B = {n1:1, n2:3}
n1: 2 > 1 → alguno mayor
n2: 1 < 3 → alguno menor
→ CONCURRENTES: hubo un conflicto real que alguien tiene que resolver
```

> [!tip] Detectar el conflicto es más valioso que resolverlo mal
> Un reloj vectorial no te dice quién gana: te dice que **hubo un conflicto**. Eso permite aplicar una regla de negocio (fusionar, preguntar al usuario, aplicar prioridad) en vez de descartar una escritura por un timestamp mentiroso.
> El costo es que el vector crece con la cantidad de nodos.

### Relojes híbridos (HLC)

Combinan lo mejor de ambos: un componente físico (aproximadamente el tiempo real, legible por humanos) y uno lógico (que garantiza el orden causal). Los usan CockroachDB y YugabyteDB.

```
HLC = (tiempo_físico, contador_lógico)
Si dos eventos comparten el tiempo físico, el contador desempata preservando la causalidad.
```

### TrueTime

Google resolvió el problema con hardware: relojes atómicos y GPS en cada datacenter. La API no devuelve un instante sino un **intervalo de incertidumbre**:

```
TT.now() → [inicio, fin]   con la garantía de que el tiempo real está adentro
```

Spanner espera a que pase la incertidumbre (unos milisegundos) antes de confirmar una transacción, y con eso logra consistencia externa global. Es la única solución que realmente ordena por tiempo físico — y requiere hardware dedicado.

## 🆔 Identificadores ordenables

Un problema práctico y frecuente: querés identificadores únicos que además ordenen por tiempo de creación.

| Opción | Único | Ordenable | Problema |
|---|---|---|---|
| `UUID` v4 | ✅ | ❌ | aleatorio: fragmenta los índices B-tree |
| Autoincremento | ✅ | ✅ | punto único de coordinación |
| Timestamp | ❌ | ✅ | colisiones |
| **UUID v7** | ✅ | ✅ | depende del reloj (pero tolera deriva) |
| **Snowflake** | ✅ | ✅ | requiere asignar id de nodo |

```java
// UUID v7: 48 bits de timestamp + aleatoriedad
UUID id = UuidCreator.getTimeOrderedEpoch();   // com.github.f4b6a3:uuid-creator
```

> [!tip] UUID v7 mejora mucho el rendimiento de los índices
> Un UUID v4 aleatorio hace que cada inserción caiga en una página distinta del índice, fragmentándolo. Un identificador ordenado por tiempo inserta siempre al final, que es el caso óptimo para un B-tree. En tablas grandes la diferencia de rendimiento de escritura es notable.

> [!warning] El orden de UUID v7 es aproximado, no causal
> Ordena por el reloj del nodo que lo generó. Sirve para índices y para ordenar aproximadamente en una interfaz. **No sirve para decidir causalidad** ni para resolver conflictos.

## 🎯 Qué usar para qué

| Necesito | Uso |
|---|---|
| Medir cuánto tardó algo | `System.nanoTime()` |
| Guardar cuándo ocurrió | `Instant.now()` en UTC |
| Ordenar eventos de un mismo agregado | número de secuencia del agregado |
| Ordenar eventos de nodos distintos | reloj lógico, o partición por clave |
| Detectar conflictos de escritura | reloj vectorial, o `@Version` |
| Expirar tokens | timestamp **con tolerancia** |
| Identificador ordenable | UUID v7 |
| Orden global estricto | consenso ([[13 - Consenso y Raft]]) |

> [!tip] El atajo que resuelve el 90% de los casos
> **Particionar por clave.** Si todos los eventos de una cuenta van a la misma partición de Kafka, se procesan en orden sin necesidad de relojes de ningún tipo. Es mucho más simple que cualquier reloj lógico, y suele ser suficiente — ver [[04 - Garantias de Entrega]].

## 🧪 Tests

```java
@Test
void la_duracion_medida_nunca_es_negativa() {
    // con nanoTime está garantizado; con currentTimeMillis no
    assertTrue(medidor.medir(() -> {}).toNanos() >= 0);
}

@Test
void el_reloj_vectorial_detecta_escrituras_concurrentes() {
    var a = new RelojVectorial(Map.of("n1", 2L, "n2", 1L));
    var b = new RelojVectorial(Map.of("n1", 1L, "n2", 3L));
    assertEquals(CONCURRENTES, a.comparar(b));
}

@Test
void el_reloj_vectorial_detecta_causalidad() {
    var a = new RelojVectorial(Map.of("n1", 1L));
    var b = a.tick("n1");
    assertEquals(ANTERIOR, a.comparar(b));
}

@Test
void un_token_es_valido_dentro_de_la_tolerancia_de_reloj() {
    var token = tokenQueExpiro(hace(Duration.ofSeconds(10)));
    assertDoesNotThrow(() -> validador.validar(token));   // tolerancia de 30 s
}

@Test
void los_uuid_v7_generados_en_secuencia_quedan_ordenados() {
    var ids = IntStream.range(0, 1000).mapToObj(i -> generar()).toList();
    assertEquals(ids, ids.stream().sorted().toList());
}
```

## ⚠️ Errores Comunes

> [!warning] Los que producen bugs irreproducibles
> - **Medir duraciones con `currentTimeMillis`**: valores negativos cuando NTP ajusta.
> - **Ordenar eventos de nodos distintos por timestamp.**
> - **Last-write-wins por reloj de pared**: pierde escrituras en silencio.
> - **Expirar sin tolerancia de reloj**: rechazos aleatorios entre servicios.
> - **Guardar `LocalDateTime`** en vez de `Instant`: depende de la zona del servidor.
> - **Suponer que el reloj es monótono en una VM**: puede saltar.
> - **UUID v4 como clave primaria** en tablas grandes: fragmenta los índices.
> - **Usar UUID v7 para decidir causalidad**: solo ordena aproximadamente.

## 📌 Notas Rápidas
- Si te llevás una sola regla: **`nanoTime` para medir, `Instant` para registrar, nunca timestamps para ordenar entre nodos.**
- El paper de Leslie Lamport de 1978 (*"Time, Clocks, and the Ordering of Events in a Distributed System"*) es uno de los más citados de la computación y sigue siendo legible. Vale la media hora.
- La razón por la que Spanner necesita relojes atómicos es exactamente esta nota: sin hardware especializado, no hay forma de ordenar por tiempo físico con garantías.
- Antes de implementar relojes lógicos, preguntate si podés particionar por clave. Casi siempre podés.

## 🔗 Relaciones
- **Módulo**: [[Sistemas Distribuidos]]
- **Anterior**: [[10 - Locks y Bloqueo Optimista]]
- **Siguiente**: [[12 - Particionado y Sharding]]
- **Se conecta con**: [[07 - java.time]], [[04 - Garantias de Entrega]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

