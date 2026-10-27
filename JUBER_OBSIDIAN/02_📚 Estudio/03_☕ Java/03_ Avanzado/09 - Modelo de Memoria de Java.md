---
tags:
  - Java
  - Avanzado
  - Concurrencia
Creado: 2026-09-08
Relacionado:
  - Java Avanzado
  - 01 - Concurrencia Threads y Runnable
Palabras clave:
  - JMM, happens-before, volatile, visibilidad, reordenamiento, publicacion segura, final
---

## 🧠 Concepto

El **modelo de memoria de Java** (JMM) define qué garantiza el lenguaje cuando varios hilos leen y escriben las mismas variables. Y la respuesta sorprende: **por defecto no garantiza casi nada**.

```java
public class Contador {
    private boolean corriendo = true;
    private int valor = 0;

    public void detener() { corriendo = false; }

    public void trabajar() {
        while (corriendo) { valor++; }     // ⚠️ puede no terminar NUNCA
    }
}
```

> [!danger] Ese bucle puede ser infinito aunque otro hilo llame a `detener()`
> No es un bug de la JVM: es comportamiento **permitido**. Sin sincronización, el compilador puede transformar el bucle en `while (true)` porque desde la perspectiva de ese hilo `corriendo` nunca cambia.
> Y lo peor: funciona en desarrollo (donde el JIT no optimizó todavía) y falla en producción bajo carga.

## 🔍 Los tres problemas

| Problema | Qué significa |
|---|---|
| **Visibilidad** | un hilo no ve los cambios que hizo otro |
| **Atomicidad** | una operación que parece indivisible no lo es |
| **Reordenamiento** | las instrucciones no se ejecutan en el orden escrito |

### Visibilidad

Cada núcleo tiene su propia caché. Una escritura puede quedar ahí y no llegar a memoria principal en un tiempo indeterminado.

```
Hilo A (núcleo 1)          Hilo B (núcleo 2)
corriendo = false
  → queda en caché L1      lee corriendo → true (su propia caché) ❌
```

### Atomicidad

```java
valor++;
// En realidad son tres operaciones:
//   1 · leer valor
//   2 · sumar 1
//   3 · escribir valor
// Dos hilos pueden intercalarse entre medio
```

```java
long dinero = 0;   // ⚠️ en JVM de 32 bits, escribir un long NO es atómico
```

> [!warning] `long` y `double` sin `volatile` pueden leerse a medias
> La especificación permite que una escritura de 64 bits se haga en dos mitades. Otro hilo puede leer la mitad vieja y la mitad nueva: un valor que **nunca existió**. En JVM de 64 bits no pasa en la práctica, pero el modelo lo permite.

### Reordenamiento

```java
// Lo que escribiste          Lo que puede ejecutarse
a = 1;                        b = 2;
b = 2;                        a = 1;
```

El compilador, el JIT y el procesador pueden reordenar mientras el resultado sea el mismo **para un solo hilo**. Con varios hilos, ese reordenamiento se vuelve observable.

## ⛓️ Happens-before

La única garantía real que da el JMM. Si la acción A *happens-before* la acción B, entonces todo lo que A hizo es **visible** para B.

### Las reglas

| Regla | Garantía |
|---|---|
| **Orden de programa** | dentro de un hilo, lo anterior *happens-before* lo posterior |
| **Monitor** | liberar un candado *happens-before* adquirirlo después |
| **`volatile`** | escribir *happens-before* leer después |
| **`Thread.start()`** | *happens-before* todo lo que haga el hilo nuevo |
| **`Thread.join()`** | todo lo del hilo *happens-before* el retorno del `join` |
| **Transitividad** | si A→B y B→C, entonces A→C |

```java
// Sin relación happens-before: sin garantías
int x = 0; boolean listo = false;

// Hilo A                  // Hilo B
x = 42;                    if (listo) {
listo = true;                  usar(x);    // ⚠️ puede ver x = 0
                           }
```

```java
// Con volatile: se establece happens-before
int x = 0; volatile boolean listo = false;

// Hilo A                  // Hilo B
x = 42;                    if (listo) {         // lee volatile
listo = true;  // volatile     usar(x);          // ✅ ve x = 42, garantizado
                           }
```

> [!tip] `volatile` publica también lo que se escribió **antes**
> No solo hace visible la variable marcada: todo lo escrito antes de la escritura volátil es visible para quien lea esa variable después. Es la propiedad que hace útil el patrón de arriba.

## 🔒 `volatile` vs `synchronized`

| | `volatile` | `synchronized` |
|---|---|---|
| Visibilidad | ✅ | ✅ |
| Evita reordenamiento | ✅ | ✅ |
| **Atomicidad de operaciones compuestas** | ❌ | ✅ |
| Exclusión mutua | ❌ | ✅ |
| Costo | bajo | mayor |

```java
// ❌ volatile NO hace atómico el incremento
private volatile int contador = 0;
public void incrementar() { contador++; }     // sigue habiendo carrera

// ✅ Opción 1: synchronized
public synchronized void incrementar() { contador++; }

// ✅ Opción 2: atómico sin candado (más rápido)
private final AtomicInteger contador = new AtomicInteger();
public void incrementar() { contador.incrementAndGet(); }
```

> [!danger] El error más común con `volatile`
> Creer que hace atómicas las operaciones. `volatile` resuelve **visibilidad y orden**, no exclusión mutua. Para leer-modificar-escribir hace falta `synchronized`, un `Atomic*` o un `Lock`.
>
> `volatile` es correcto solo cuando el nuevo valor **no depende del anterior**: una bandera de parada, una referencia que se reemplaza entera.

## 📦 Publicación segura

Publicar un objeto es hacerlo visible a otros hilos. Hacerlo mal es una fuente clásica de bugs irreproducibles.

```java
// ❌ Publicación insegura: otro hilo puede ver el objeto A MEDIO CONSTRUIR
public class Servicio {
    public static Config config;

    public static void inicializar() {
        config = new Config();     // el constructor puede reordenarse con la asignación
    }
}
```

```
Lo que puede pasar:
  1 · se asigna la referencia
  2 · otro hilo la lee y usa el objeto
  3 · recién ahí termina el constructor  💥
```

### Las formas seguras

```java
// 1 · Inicialización estática: la JVM lo garantiza
private static final Config CONFIG = new Config();

// 2 · volatile
private static volatile Config config;

// 3 · Campos final (con this sin escapar del constructor)
public final class Config {
    private final String url;
    public Config(String url) { this.url = url; }    // los final se publican seguros
}

// 4 · Holder idiom: perezoso y seguro sin sincronización
private static class Holder {
    static final Config INSTANCIA = new Config();
}
public static Config obtener() { return Holder.INSTANCIA; }
```

> [!tip] Los campos `final` tienen garantías especiales
> Si un objeto solo tiene campos `final` y `this` no escapa del constructor, cualquier hilo que vea la referencia ve los campos **completamente inicializados**, sin necesidad de sincronización.
> Es una de las razones por las que la inmutabilidad simplifica tanto la concurrencia — ver [[10 - equals hashCode e Inmutabilidad]].

> [!danger] `this` no debe escapar del constructor
> ```java
> public class Servicio {
>     public Servicio(Registro r) {
>         r.registrar(this);        // ❌ otro hilo ve el objeto incompleto
>         this.config = cargar();   // esto todavía no pasó
>     }
> }
> ```
> Registrar `this`, iniciar un hilo o pasar `this` a un callback dentro del constructor rompe las garantías de publicación segura.

## 🧊 Inmutabilidad: la salida más simple

```java
public record Dinero(BigDecimal monto, Moneda moneda) { }
```

Un objeto inmutable **no tiene problemas de concurrencia**: no hay escrituras que puedan verse a medias, no hay estado compartido que proteger, no hace falta sincronización.

> [!tip] La regla práctica que evita el 90% de los bugs de concurrencia
> **Preferí inmutabilidad. Si hay estado mutable, no lo compartas. Si tenés que compartirlo, sincronizalo.**
> En ese orden. La mayoría de los problemas desaparecen antes del tercer paso.

## 🎯 Herramientas, en orden de preferencia

```
1 · Inmutabilidad                     → sin problema que resolver
2 · Confinamiento a un hilo           → ThreadLocal, variables locales
3 · Colecciones concurrentes          → ConcurrentHashMap, CopyOnWriteArrayList
4 · Clases atómicas                   → AtomicInteger, AtomicReference, LongAdder
5 · volatile                          → banderas, referencias que se reemplazan
6 · synchronized / Lock               → operaciones compuestas
7 · Sincronización manual con wait()  → casi nunca hace falta
```

```java
// LongAdder gana a AtomicLong con mucha contención
private final LongAdder peticiones = new LongAdder();
peticiones.increment();      // reparte el conteo en celdas: menos contención
long total = peticiones.sum();
```

## 🧪 Cómo se detectan estos bugs

```java
@Test
void el_contador_no_pierde_incrementos() {
    var contador = new Contador();
    var listos = new CountDownLatch(1);

    var futuros = IntStream.range(0, 100)
        .mapToObj(i -> pool.submit(() -> {
            listos.await();                      // arrancan todos juntos
            for (int j = 0; j < 1000; j++) contador.incrementar();
            return null;
        })).toList();

    listos.countDown();
    futuros.forEach(this::esperar);

    assertEquals(100_000, contador.valor());     // con ++ sin sincronizar, falla
}
```

> [!warning] Estos tests fallan de forma intermitente
> Un test de concurrencia que pasa no prueba que el código sea correcto: prueba que esa vez no se manifestó el problema. Hay que ejecutarlos muchas veces y con carga.
> **jcstress** (del proyecto OpenJDK) es la herramienta seria para esto: fuerza reordenamientos y explora entrelazados que en un test normal casi nunca aparecen.

```bash
# Detección de condiciones de carrera con ThreadSanitizer (experimental)
java -XX:+UseZGC -Xcheck:jni ...

# Analizadores estáticos
mvn spotbugs:check      # detecta varios patrones de concurrencia incorrecta
```

## ⚠️ Errores Comunes

> [!warning] Los que producen bugs irreproducibles
> - **Creer que `volatile` hace atómico** el incremento.
> - **Bucle sobre una bandera no volátil**: puede no terminar nunca.
> - **Publicar `this` desde el constructor.**
> - **Doble verificación sin `volatile`** en un singleton perezoso.
> - **Estado mutable en un bean singleton** de Spring: compartido entre todos los requests.
> - **`SimpleDateFormat` como campo estático**: no es seguro para hilos.
> - **Suponer que "funciona en mi máquina" prueba algo**: el JIT optimiza distinto bajo carga.
> - **Sincronizar sobre `String` o `Integer`**: son objetos internados y compartidos.

```java
// ❌ El singleton con doble verificación SIN volatile está roto
private static Config instancia;
public static Config obtener() {
    if (instancia == null) {
        synchronized (Config.class) {
            if (instancia == null) instancia = new Config();  // reordenamiento posible
        }
    }
    return instancia;
}

// ✅ Con volatile, o mejor: usar el holder idiom
private static volatile Config instancia;
```

## 📌 Notas Rápidas
- El JMM es de los temas que más separan a un senior en una entrevista, porque casi nadie lo estudia: la mayoría usa `synchronized` por costumbre sin saber qué garantiza.
- La frase que hay que poder decir: **"`volatile` da visibilidad y orden, no atomicidad"**. Con eso y un ejemplo, se demuestra que entendiste el modelo.
- Los bugs de este tipo son los más caros de diagnosticar: no se reproducen, no dejan stacktrace y suelen manifestarse como datos incorrectos, no como excepciones.
- La especificación es JSR-133. El libro de referencia es *Java Concurrency in Practice*, que sigue vigente veinte años después.

## 🔗 Relaciones
- **Módulo**: [[Java Avanzado]]
- **Base**: [[01 - Concurrencia Threads y Runnable]], [[02 - ExecutorService y Thread Pools]]
- **Siguiente**: [[10 - equals hashCode e Inmutabilidad]]
- **Se conecta con**: [[10 - Locks y Bloqueo Optimista]], [[08 - Virtual Threads]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

