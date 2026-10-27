---
tags:
  - Java
  - Avanzado
  - Colecciones
Creado: 2026-09-08
Relacionado:
  - Java Avanzado
  - 08 - Arrays y Colecciones
Palabras clave:
  - HashMap, ArrayList, treeify, factor de carga, redimensionado, ConcurrentHashMap
---

## 🧠 Concepto

Elegir una colección "porque es la que se usa siempre" funciona hasta que el volumen crece. Entender cómo están implementadas convierte esa elección en una decisión con fundamento — y explica un montón de comportamientos que de otro modo parecen magia.

## 📋 `ArrayList`

```java
transient Object[] elementData;      // un array plano
private int size;
```

| Operación | Complejidad | Comentario |
|---|---|---|
| `get(i)` | **O(1)** | acceso directo por índice |
| `add(e)` al final | O(1) amortizado | salvo cuando toca crecer |
| `add(i, e)` al medio | O(n) | desplaza todo lo posterior |
| `remove(i)` | O(n) | ídem |
| `contains(e)` | O(n) | recorre linealmente |

### El crecimiento

```java
// Cuando se llena: nuevo array un 50% más grande y copia completa
int nuevaCapacidad = viejaCapacidad + (viejaCapacidad >> 1);   // ×1,5
elementData = Arrays.copyOf(elementData, nuevaCapacidad);
```

```
Capacidad inicial 10 → 15 → 22 → 33 → 49 → 73 → 109 ...
Cada crecimiento copia TODO el array.
```

> [!tip] Si sabés el tamaño, declaralo
> ```java
> var lista = new ArrayList<Movimiento>();          // 10 → varias copias
> var lista = new ArrayList<Movimiento>(50_000);    // ✅ una sola reserva
> ```
> Llenar una lista de 100.000 elementos sin capacidad inicial hace ~25 redimensionados y copia millones de referencias. Es de las optimizaciones más baratas que existen.

> [!warning] `ArrayList` guarda referencias, no valores
> Una lista de un millón de `Integer` son un millón de objetos en el heap más el array de referencias: unas 20 veces más memoria que un `int[]`. Para colecciones numéricas grandes, un array primitivo o una biblioteca especializada (Eclipse Collections, fastutil) hace una diferencia real.

## 🗺️ `HashMap` — el más importante

```java
Node<K,V>[] table;         // array de buckets
static final int TREEIFY_THRESHOLD = 8;
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

### Cómo encuentra un elemento

```java
// 1 · Mezclar el hash: propaga los bits altos a los bajos
static final int hash(Object key) {
    int h = key.hashCode();
    return h ^ (h >>> 16);
}

// 2 · Índice del bucket (n es potencia de 2)
int indice = (n - 1) & hash;      // equivalente a hash % n, pero mucho más rápido
```

> [!tip] Por qué el `^ (h >>> 16)`
> El índice se calcula con `(n-1) & hash`, que **solo usa los bits bajos**. Si dos claves difieren solo en los bits altos, caerían en el mismo bucket. Mezclar el hash consigo mismo desplazado propaga la información de los bits altos hacia abajo y reparte mucho mejor.
> Es la razón por la que un `hashCode` mediocre no arruina tanto el rendimiento como uno esperaría.

### La resolución de colisiones

```
Bucket 3: [nodo] → [nodo] → [nodo]        lista enlazada: O(n) en el bucket
                    ↓ con 8+ nodos Y tabla ≥ 64
Bucket 3: árbol rojo-negro                O(log n)
```

| Umbral | Valor | Qué hace |
|---|---|---|
| `TREEIFY_THRESHOLD` | 8 | el bucket pasa a árbol |
| `UNTREEIFY_THRESHOLD` | 6 | vuelve a lista |
| `MIN_TREEIFY_CAPACITY` | 64 | por debajo, prefiere redimensionar |

> [!tip] La conversión a árbol (Java 8+) es una defensa de seguridad
> Antes, un atacante podía enviar miles de claves con el mismo hash y degradar el `HashMap` a una lista enlazada: cada búsqueda O(n) y el servidor colapsaba. Es el ataque de **colisiones de hash**.
> Con el árbol, el peor caso pasa de O(n) a O(log n) y el ataque deja de ser viable.

### El redimensionado

```
Cuando size > capacidad × 0,75 → capacidad × 2, y se redistribuye TODO
```

```java
// Sin capacidad inicial: 10.000 elementos causan ~10 redimensionados
var mapa = new HashMap<String, Cuenta>();

// ✅ Con capacidad calculada: ninguno
var mapa = new HashMap<String, Cuenta>((int)(10_000 / 0.75f) + 1);
```

> [!warning] El factor de carga 0,75 es un equilibrio, no un capricho
> Más bajo (0,5): menos colisiones, más memoria desperdiciada. Más alto (0,95): menos memoria, más colisiones. 0,75 es el punto donde la probabilidad de colisión y el uso de memoria se equilibran razonablemente para el caso general.

> [!danger] `null` como clave: solo uno, y solo en `HashMap`
> `HashMap` admite una clave `null` (va al bucket 0, sin llamar a `hashCode`). `ConcurrentHashMap` y `TreeMap` **lanzan `NullPointerException`**. Es una diferencia que sorprende al migrar de una a otra.

## 🌳 `TreeMap`

Árbol rojo-negro: mantiene las claves ordenadas.

| Operación | Complejidad |
|---|---|
| `get`, `put`, `remove` | O(log n) |
| `firstKey`, `lastKey` | O(log n) |
| `subMap`, `headMap`, `tailMap` | O(log n) |
| Recorrido ordenado | O(n) |

```java
var porFecha = new TreeMap<LocalDate, Saldo>();
porFecha.floorEntry(fecha);            // el saldo más cercano ANTERIOR o igual ✅
porFecha.subMap(desde, true, hasta, true);
```

> [!tip] `floorEntry` es exactamente lo que necesita el saldo histórico
> Buscar "el snapshot más cercano antes de esta fecha" es una operación natural de `TreeMap` y O(log n). Es la estructura correcta para índices por rango — ver [[C7 - Estado de Cuenta]].

## 🔗 Las variantes ordenadas

| | Orden | Costo extra |
|---|---|---|
| `HashMap` | ninguno | — |
| `LinkedHashMap` | de inserción (o de acceso) | lista doblemente enlazada |
| `TreeMap` | natural o por comparador | O(log n) en todo |

```java
// Caché LRU en cuatro líneas
var cache = new LinkedHashMap<String, Datos>(16, 0.75f, true) {   // true = orden de acceso
    @Override protected boolean removeEldestEntry(Map.Entry<String, Datos> eldest) {
        return size() > 1000;
    }
};
```

## ⚡ `ConcurrentHashMap`

```java
// ❌ Un HashMap con varios hilos puede CORROMPERSE, no solo dar datos raros
var mapa = new HashMap<String, Cuenta>();

// ✅
var mapa = new ConcurrentHashMap<String, Cuenta>();
```

> [!danger] Un `HashMap` bajo concurrencia puede entrar en bucle infinito
> No es una exageración: durante un redimensionado concurrente en Java 7, la lista enlazada de un bucket podía quedar circular, y una búsqueda posterior no terminaba nunca. El síntoma era un núcleo al 100% sin explicación.
> En Java 8 el mecanismo cambió y ese caso concreto no ocurre, pero **la clase sigue sin ser segura para hilos**: se pierden entradas y el estado queda inconsistente.

**Cómo lo resuelve `ConcurrentHashMap` (desde Java 8):**

| Operación | Mecanismo |
|---|---|
| Lectura | **sin candado** (campos `volatile`) |
| Escritura en bucket vacío | CAS (compare-and-swap) |
| Escritura en bucket ocupado | `synchronized` sobre el primer nodo del bucket |

```
La granularidad del candado es el BUCKET, no el mapa.
Dos escrituras en buckets distintos no se bloquean entre sí.
```

> [!tip] Las operaciones atómicas son el motivo principal para usarlo
> ```java
> // ❌ Dos operaciones: hay ventana de carrera entre ellas
> if (!mapa.containsKey(k)) mapa.put(k, calcular(k));
>
> // ✅ Atómico: solo un hilo ejecuta el cálculo
> mapa.computeIfAbsent(k, this::calcular);
> ```
> `computeIfAbsent`, `merge`, `putIfAbsent` y `compute` son atómicas. Componer operaciones a mano no lo es.

> [!warning] No hagas operaciones costosas dentro de `computeIfAbsent`
> El bucket queda bloqueado durante la función. Una llamada remota ahí bloquea a todos los que toquen ese bucket. Y si la función accede al mismo mapa, puede producir un interbloqueo.

## 📝 `CopyOnWriteArrayList`

```java
// Cada escritura copia el array COMPLETO
public boolean add(E e) {
    Object[] nuevo = Arrays.copyOf(elementos, elementos.length + 1);
    nuevo[elementos.length] = e;
    elementos = nuevo;      // volatile: publicación segura
}
```

| Lecturas | Escrituras | Uso ideal |
|---|---|---|
| sin candado, rapidísimas | O(n), copian todo | listas de escuchas, configuración |

> [!danger] Inútil si hay escrituras frecuentes
> Con 1.000 elementos y 100 escrituras por segundo, copiás 100.000 referencias por segundo. Solo tiene sentido cuando las lecturas dominan abrumadoramente y las escrituras son excepcionales.

## 🎯 Guía de elección

| Necesito | Uso | Por qué |
|---|---|---|
| Lista con acceso por índice | `ArrayList` | O(1) y memoria contigua |
| Cola FIFO o pila | `ArrayDeque` | más rápida que `LinkedList` y que `Stack` |
| Muchas inserciones al principio | `ArrayDeque` o `LinkedList` | O(1) en los extremos |
| Sin duplicados | `HashSet` | O(1) |
| Sin duplicados, ordenado | `TreeSet` | O(log n) |
| Sin duplicados, orden de inserción | `LinkedHashSet` | O(1) + orden |
| Clave-valor | `HashMap` | O(1) |
| Clave-valor ordenado o por rangos | `TreeMap` | `floorEntry`, `subMap` |
| Concurrente | `ConcurrentHashMap` | sin candado en lectura |
| Muchas lecturas, casi sin escrituras | `CopyOnWriteArrayList` | lectura sin sincronizar |
| Cola entre productor y consumidor | `LinkedBlockingQueue` **acotada** | backpressure |

> [!warning] `LinkedList` casi nunca es la respuesta
> Su ventaja teórica (O(1) al insertar al medio) requiere ya tener el iterador en esa posición; si hay que llegar con `get(i)`, es O(n) y perdiste. Además cada elemento es un objeto extra con dos punteros: peor localidad de caché y más memoria.
> **En la práctica, `ArrayList` gana casi siempre**, incluso en escenarios donde la teoría diría lo contrario.

## 🚫 Inmutables y vistas

```java
List.of("a", "b");                    // inmutable de verdad, rechaza null
Collections.unmodifiableList(lista);  // ⚠️ VISTA: si cambia el original, cambia la vista
List.copyOf(lista);                   // ✅ copia inmutable independiente
```

> [!danger] `unmodifiableList` no protege del todo
> ```java
> var original = new ArrayList<>(List.of("a"));
> var vista = Collections.unmodifiableList(original);
> original.add("b");
> vista.size();      // → 2  💥 la "inmutable" cambió
> ```
> Es una vista de solo lectura, no una copia. Para inmutabilidad real: `List.copyOf` — ver la copia defensiva de [[10 - equals hashCode e Inmutabilidad]].

## 💥 `ConcurrentModificationException`

```java
// ❌ Modificar mientras se recorre
for (var s : lista) if (s.startsWith("x")) lista.remove(s);

// ✅
lista.removeIf(s -> s.startsWith("x"));
```

> [!tip] No siempre es un problema de hilos
> El nombre confunde: ocurre con un solo hilo. La colección lleva un contador de modificaciones (`modCount`) y el iterador verifica que no haya cambiado. Es una **detección de bug**, no un mecanismo de sincronización — de hecho no está garantizada, solo es de "mejor esfuerzo".

## 🧪 Medir

```java
@Test
void reservar_capacidad_evita_redimensionados() {
    var sin = medir(() -> { var l = new ArrayList<Integer>();
                            for (int i = 0; i < 1_000_000; i++) l.add(i); });
    var con = medir(() -> { var l = new ArrayList<Integer>(1_000_000);
                            for (int i = 0; i < 1_000_000; i++) l.add(i); });
    assertTrue(con.compareTo(sin) < 0);
}
```

Para mediciones serias, **JMH** — ver [[06 - Benchmarking con JMH]]. Un `System.nanoTime()` alrededor de un bucle mide sobre todo el calentamiento del JIT, no el código.

## ⚠️ Errores Comunes

> [!warning] Los que cuestan memoria o corrección
> - **`HashMap` compartido entre hilos**: estado corrupto.
> - **Componer operaciones** en vez de usar `computeIfAbsent` o `merge`.
> - **Sin capacidad inicial** en colecciones grandes conocidas.
> - **`LinkedList` "porque inserta rápido"**: casi nunca gana.
> - **`unmodifiableList`** creyendo que es una copia.
> - **Objeto mutable como clave**: se pierde en el mapa.
> - **Modificar durante el recorrido** en vez de `removeIf`.
> - **`CopyOnWriteArrayList` con escrituras frecuentes.**
> - **Colas ilimitadas** en pools de hilos → ver [[06 - Colas y Backpressure]].

## 📌 Notas Rápidas
- El detalle que más impresiona en una entrevista: **por qué el `HashMap` convierte buckets en árboles**. La respuesta correcta no es "por rendimiento" sino "para mitigar ataques de colisiones de hash".
- Reservar la capacidad inicial es la optimización con mejor relación esfuerzo/beneficio en código que llena colecciones grandes.
- El bucle infinito del `HashMap` en Java 7 bajo concurrencia es una anécdota real que explica muy bien por qué "no seguro para hilos" no significa "a veces da datos raros".
- Todo lo de esta nota depende de que `hashCode` esté bien implementado — ver [[10 - equals hashCode e Inmutabilidad]].

## 🔗 Relaciones
- **Módulo**: [[Java Avanzado]]
- **Anterior**: [[10 - equals hashCode e Inmutabilidad]]
- **Siguiente**: [[12 - Structured Concurrency]]
- **Base**: [[08 - Arrays y Colecciones]]
- **Se conecta con**: [[04 - JVM Memoria y GC]], [[06 - Benchmarking con JMH]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

