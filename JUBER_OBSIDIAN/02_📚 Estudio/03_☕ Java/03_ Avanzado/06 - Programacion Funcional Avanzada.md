---
tags:
  - Java
  - Avanzado
Creado: 2026-05-21
Relacionado:
  - Java Avanzado
  - Lambdas y Functional Interfaces
  - Streams API
Palabras clave:
  - monad, composición, currying, partial application, trampolining, collector custom
---

## 🧠 Composición de funciones

```java
Function<Integer, Integer> doble = x -> x * 2;
Function<Integer, Integer> masUno = x -> x + 1;

// andThen — primero doble, luego masUno
Function<Integer, Integer> dobleMasUno = doble.andThen(masUno);
dobleMasUno.apply(3); // 7

// compose — primero masUno, luego doble
Function<Integer, Integer> masUnoDoble = doble.compose(masUno);
masUnoDoble.apply(3); // 8
```

## Currying y Partial Application

```java
// Currying: transformar f(a, b) en f(a)(b)
Function<Integer, Function<Integer, Integer>> sumarCurry = a -> b -> a + b;
Function<Integer, Integer> sumar5 = sumarCurry.apply(5);
sumar5.apply(3);  // 8
sumar5.apply(10); // 15

// Partial application — fijar algunos argumentos
BiFunction<String, Integer, String> repetir = (s, n) -> s.repeat(n);
Function<Integer, String> repetirJava = n -> repetir.apply("Java", n);
repetirJava.apply(3); // "JavaJavaJava"
```

## Collectors personalizados

```java
// Collector que concatena strings con separador
Collector<String, StringJoiner, String> joinCollector = Collector.of(
    () -> new StringJoiner(", ", "[", "]"),    // supplier
    StringJoiner::add,                          // accumulator
    StringJoiner::merge,                        // combiner (para paralelo)
    StringJoiner::toString                      // finisher
);

List.of("Ana", "Juber", "Carlos").stream()
    .collect(joinCollector); // "[Ana, Juber, Carlos]"
```

## Memoización — cachear resultados

```java
public static <T, R> Function<T, R> memoize(Function<T, R> funcion) {
    Map<T, R> cache = new ConcurrentHashMap<>();
    return input -> cache.computeIfAbsent(input, funcion);
}

Function<Integer, Long> fibonacci = memoize(n -> {
    if (n <= 1) return (long) n;
    return fibonacci.apply(n - 1) + fibonacci.apply(n - 2);
});
```

## Lazy evaluation con Supplier

```java
// Sin lazy — se evalúa inmediatamente
String resultado = operacionCostosa(); // se ejecuta siempre

// Con lazy — se evalúa solo cuando se necesita
Supplier<String> lazy = () -> operacionCostosa();
// ...
if (condicion) {
    String resultado = lazy.get(); // solo se ejecuta aquí
}
```

## Pattern Matching con instanceof (Java 16+)

```java
// Antes
if (objeto instanceof String) {
    String s = (String) objeto;
    System.out.println(s.length());
}

// Con pattern matching
if (objeto instanceof String s) {
    System.out.println(s.length()); // s disponible directamente
}

// En switch (Java 21)
String descripcion = switch (objeto) {
    case Integer i -> "Entero: " + i;
    case String s when s.length() > 5 -> "String largo: " + s;
    case String s -> "String corto: " + s;
    case null -> "Nulo";
    default -> "Otro: " + objeto;
};
```

## Streams avanzados — teeing (Java 12+)

```java
// Dos collectors en uno
var resultado = Stream.of(1, 2, 3, 4, 5, 6)
    .collect(Collectors.teeing(
        Collectors.filtering(n -> n % 2 == 0, Collectors.toList()), // pares
        Collectors.filtering(n -> n % 2 != 0, Collectors.toList()), // impares
        (pares, impares) -> Map.of("pares", pares, "impares", impares)
    ));
```

## Operaciones en paralelo seguras

```java
// ✅ Reducción asociativa y sin estado — segura en paralelo
List<Integer> numeros = IntStream.rangeClosed(1, 1_000_000)
    .boxed()
    .collect(Collectors.toList());

long suma = numeros.parallelStream()
    .mapToLong(Integer::longValue)
    .sum(); // thread-safe

// ❌ Efectos secundarios con parallelStream — inseguro
List<Integer> resultado = new ArrayList<>();
numeros.parallelStream().forEach(resultado::add); // race condition
```

## ⚠️ Errores Comunes

> [!warning] No usar `forEach` con `parallelStream` sobre colecciones no thread-safe
> Usar `collect(Collectors.toList())` en su lugar.

## 📌 Notas Rápidas
- La composición de funciones (`Function.andThen`, `Function.compose`) permite construir pipelines de transformación reutilizables sin escribir un método por cada combinación.
- La inmutabilidad (records, `List.of()`, `Collections.unmodifiableList()`) es la base que hace segura la programación funcional en un lenguaje con estado mutable como Java.
- `Collectors` personalizados (`Collector.of(...)`) permiten construir reducciones custom más allá de `toList()`/`groupingBy()`, para casos que no cubre la librería estándar.
- El estilo funcional favorece expresividad sobre performance bruta en casos simples — para hot paths críticos, medir antes de asumir que un stream es igual de rápido que un bucle imperativo.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

