---
tags:
  - Java
  - Fundamentos
Creado: 2026-09-09
Relacionado:
  - Java Fundamentos
  - 01 - Variables y Tipos de Datos
Palabras clave:
  - wrapper, autoboxing, unboxing, Integer, cache, parseInt, valueOf, NullPointerException
---

## 🧠 Concepto

Java tiene **dos mundos separados**: los tipos primitivos (`int`, `double`, `boolean`…) y los objetos. Los primitivos son rápidos y viven en la pila, pero no son objetos: no tienen métodos, no pueden ser `null` y **no entran en las colecciones**.

Las **clases wrapper** son la versión objeto de cada primitivo. Envuelven el valor para que se pueda tratar como cualquier otro objeto.

```java
int primitivo = 42;              // un valor
Integer objeto = 42;             // un objeto que CONTIENE el valor 42

primitivo.toString();            // ❌ no compila: un int no tiene métodos
objeto.toString();               // ✅ "42"

List<int> lista;                 // ❌ no compila
List<Integer> lista;             // ✅
```

## 📦 Los ocho wrappers

| Primitivo | Wrapper | Hereda de |
|---|---|---|
| `byte` | `Byte` | `Number` |
| `short` | `Short` | `Number` |
| `int` | **`Integer`** | `Number` |
| `long` | `Long` | `Number` |
| `float` | `Float` | `Number` |
| `double` | `Double` | `Number` |
| `char` | `Character` | `Object` |
| `boolean` | `Boolean` | `Object` |

> [!tip] Los nombres no son todos obvios
> `int` → `Integer` y `char` → `Character` son los dos que no siguen el patrón de "misma palabra en mayúscula". Es un error de tipeo frecuente al empezar.

## 🔄 Autoboxing y unboxing

Desde Java 5 el compilador hace la conversión solo. **No es magia: es azúcar sintáctico** que inserta llamadas por vos.

```java
// Lo que escribís                    Lo que el compilador genera
Integer a = 42;                       Integer a = Integer.valueOf(42);   // autoboxing
int b = a;                            int b = a.intValue();              // unboxing

List<Integer> lista = new ArrayList<>();
lista.add(5);                         lista.add(Integer.valueOf(5));
int primero = lista.get(0);           int primero = lista.get(0).intValue();
```

| Dirección | Nombre | Qué inserta |
|---|---|---|
| primitivo → objeto | **autoboxing** | `Wrapper.valueOf(x)` |
| objeto → primitivo | **unboxing** | `x.intValue()`, `x.doubleValue()`… |

## 💥 El bug número uno: unboxing de `null`

```java
Map<String, Boolean> permisos = new HashMap<>();

if (permisos.get("editar")) {        // 💥 NullPointerException
    ...
}
```

La clave no existe, `get` devuelve `null`, y para evaluar el `if` el compilador inserta `.booleanValue()` **sobre `null`**.

> [!danger] Es el bug más silencioso de los wrappers
> El código se lee perfectamente natural y no hay ningún `.` visible que delate la llamada. El NPE aparece en una línea donde aparentemente solo comparás un booleano.

```java
// ✅ Manejar la ausencia explícitamente
if (Boolean.TRUE.equals(permisos.get("editar"))) { ... }
if (permisos.getOrDefault("editar", false)) { ... }
```

```java
// Otro caso clásico
public int calcular(Integer valor) {
    return valor * 2;                 // 💥 NPE si valor es null
}
```

## 🎯 El caché de `Integer`

Acá está el comportamiento que confunde a todo el mundo.

```java
Integer a = 127, b = 127;
a == b;                               // true   ✅

Integer c = 128, d = 128;
c == d;                               // false  💥 mismo valor, distinto objeto
```

**Por qué:** `Integer.valueOf()` mantiene un caché de objetos ya creados para el rango **-128 a 127**. Dentro de ese rango devuelve siempre la misma instancia; fuera, crea una nueva.

```java
// Lo que hace valueOf por dentro, simplificado
public static Integer valueOf(int i) {
    if (i >= -128 && i <= IntegerCache.high) return IntegerCache.cache[i + 128];
    return new Integer(i);
}
```

| Wrapper | ¿Tiene caché? | Rango |
|---|---|---|
| `Byte`, `Short`, `Integer`, `Long` | ✅ | -128 a 127 |
| `Character` | ✅ | 0 a 127 |
| `Boolean` | ✅ | `TRUE` y `FALSE` |
| **`Float`, `Double`** | ❌ | nunca |

> [!danger] La conclusión práctica: nunca compares wrappers con `==`
> ```java
> if (idA == idB)              // ❌ compara referencias: funciona "de casualidad" bajo 128
> if (idA.equals(idB))         // ✅
> if (idA.intValue() == idB.intValue())   // ✅ compara valores
> ```
> El bug es especialmente traicionero porque **funciona en las pruebas** —donde los ids suelen ser 1, 2, 3— y falla en producción cuando los ids superan 127.

## 🏗️ Crear wrappers

```java
Integer.valueOf(42);          // ✅ usa el caché
Integer.valueOf("42");        // ✅ desde String, devuelve Integer
Integer.parseInt("42");       // ✅ desde String, devuelve int primitivo

new Integer(42);              // ❌ obsoleto desde Java 9, eliminado: nunca usa el caché
```

| Método | Devuelve | Cuándo |
|---|---|---|
| `Integer.parseInt(s)` | `int` | vas a hacer cálculos |
| `Integer.valueOf(s)` | `Integer` | vas a guardarlo en una colección |
| `Integer.valueOf(i)` | `Integer` | boxing explícito |

```java
// parseInt lanza excepción si no es un número
Integer.parseInt("abc");      // 💥 NumberFormatException
Integer.parseInt("");         // 💥
Integer.parseInt(null);       // 💥

// Validar antes, o capturar
try { return Integer.parseInt(s); }
catch (NumberFormatException e) { return valorPorDefecto; }
```

## ⚖️ Cuándo usar cada uno

| Usá primitivo | Usá wrapper |
|---|---|
| cálculos y bucles | colecciones (`List<Integer>`) |
| contadores | campos que pueden ser `null` |
| campos de alto volumen | genéricos (`Optional<Integer>`) |
| parámetros obligatorios | parámetros opcionales |
| — | claves de mapas |

```java
// ✅ El wrapper comunica que el dato PUEDE faltar
public record ClienteRequest(
        String nombre,
        int edad,                     // obligatorio
        Integer telefono              // opcional: null significa "no informado"
) {}
```

> [!tip] Un `Integer` que nunca puede ser `null` es un `int` mal declarado
> El wrapper agrega la posibilidad de ausencia. Si esa posibilidad no existe en tu dominio, usar el wrapper obliga a todos los llamadores a considerar un caso imposible — y tarde o temprano alguien va a olvidarse y va a producir un NPE.

## 🐌 El costo real

```java
int[] primitivos = new int[1_000_000];              // ~4 MB
List<Integer> objetos = new ArrayList<>(1_000_000); // ~20 MB
```

Cada `Integer` es un objeto en el heap: cabecera + el valor + la referencia desde el array. **Unas 5 veces más memoria**, más presión sobre el recolector de basura.

```java
// ❌ Un millón de boxing y unboxing en el bucle
Long suma = 0L;
for (long i = 0; i < 1_000_000; i++) {
    suma += i;                        // desempaqueta, suma, vuelve a empaquetar
}

// ✅ Primitivo
long suma = 0L;
for (long i = 0; i < 1_000_000; i++) { suma += i; }
```

> [!warning] El primer bloque crea un millón de objetos `Long`
> `suma += i` sobre un `Long` no modifica el objeto (son inmutables): crea uno nuevo en cada iteración. En un bucle caliente la diferencia de rendimiento es de un orden de magnitud.
> Por eso los streams tienen `IntStream`, `LongStream` y `DoubleStream`: evitan el boxing — ver [[03 - Streams API]].

## 🎭 La trampa del ternario

```java
Object resultado = true ? Integer.valueOf(1) : Double.valueOf(2.0);
System.out.println(resultado);        // 1.0  ← ¡es un Double!
```

El operador ternario aplica **promoción numérica** cuando las dos ramas son numéricas de distinto tipo: desempaqueta ambas, las promueve al tipo más amplio y vuelve a empaquetar.

```java
Integer valor = null;
int x = true ? 0 : valor;             // ok
int x = false ? 0 : valor;            // 💥 NPE al desempaquetar null
```

> [!warning] Es un caso raro pero real
> Aparece sobre todo al mezclar tipos en expresiones condicionales dentro de mapeos. Si las dos ramas devuelven wrappers distintos, revisá qué tipo sale.

## 🧰 Métodos útiles

```java
Integer.MAX_VALUE;  Integer.MIN_VALUE;
Integer.compare(a, b);                 // sin riesgo de overflow, a diferencia de a - b
Integer.toBinaryString(10);            // "1010"
Integer.toHexString(255);              // "ff"
Integer.sum(a, b);  Integer.max(a, b);

Character.isDigit(c);  Character.isLetter(c);  Character.toUpperCase(c);

Boolean.parseBoolean("true");          // true; cualquier otra cosa es false
Boolean.TRUE.equals(posibleNull);      // patrón seguro contra null

Double.isNaN(d);  Double.isInfinite(d);
Double.compare(a, b);                  // maneja NaN correctamente
```

> [!danger] Nunca compares `double` con `==`
> ```java
> 0.1 + 0.2 == 0.3;                     // false
> Double.NaN == Double.NaN;             // false (!)
> ```
> Y para dinero, ni primitivo ni wrapper: `BigDecimal` — ver [[01 - Dinero en Software]].

## 🧪 Para probar y entender

```java
@Test
void el_cache_de_Integer_afecta_la_comparacion_con_igualdad() {
    assertSame(Integer.valueOf(127), Integer.valueOf(127));      // mismo objeto
    assertNotSame(Integer.valueOf(128), Integer.valueOf(128));   // objetos distintos
    assertEquals(Integer.valueOf(128), Integer.valueOf(128));    // mismo valor ✅
}

@Test
void desempaquetar_null_lanza_NPE() {
    Map<String, Integer> m = new HashMap<>();
    assertThrows(NullPointerException.class, () -> { int x = m.get("no-existe"); });
}

@Test
void getOrDefault_evita_el_NPE() {
    Map<String, Integer> m = new HashMap<>();
    assertEquals(0, m.getOrDefault("no-existe", 0));
}
```

## ⚠️ Errores Comunes

> [!warning] Los que se ven en código real
> - **Comparar wrappers con `==`**: funciona hasta 127 y falla después.
> - **Desempaquetar `null`**: NPE en una línea que parece inofensiva.
> - **`Integer` donde el valor nunca falta**: obliga a chequear un caso imposible.
> - **Boxing dentro de un bucle caliente**: un objeto por iteración.
> - **`a - b` para comparar**: puede desbordar. Usá `Integer.compare(a, b)`.
> - **`new Integer(...)`**: obsoleto, y nunca usa el caché.
> - **`parseInt` sin manejar `NumberFormatException`.**
> - **`==` entre `double`**: nunca es confiable.

## 📌 Notas Rápidas
- Regla de bolsillo: **primitivo por defecto, wrapper cuando necesitás `null` o una colección**.
- El caché de -128 a 127 se puede ampliar con `-XX:AutoBoxCacheMax=<n>`, pero **no lo hagas**: hace que tu código dependa de una bandera de la JVM y funcione distinto en otra máquina.
- El bug del `==` con ids es de los más comunes en entrevistas y en producción: sobrevive a todas las pruebas donde los ids son chicos.
- Saber que el autoboxing inserta `valueOf()` e `intValue()` explica de un saque el caché, el NPE de unboxing y el costo en bucles. Es una sola idea que resuelve tres preguntas.

## 🔗 Relaciones
- **Módulo**: [[Java Fundamentos]]
- **Base**: [[01 - Variables y Tipos de Datos]]
- **Se conecta con**: [[08 - Arrays y Colecciones]], [[Java-errores-comunes]], [[01 - Dinero en Software]]
- **Rendimiento**: [[03 - Streams API]], [[11 - Internals de Colecciones]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
