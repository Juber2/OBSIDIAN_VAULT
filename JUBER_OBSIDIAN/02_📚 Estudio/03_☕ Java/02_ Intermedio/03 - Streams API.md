---
tags:
  - Java
  - Intermedio
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
  - Lambdas y Functional Interfaces
Palabras clave:
  - streams, pipeline, filter, map, reduce, collect, terminal, intermedia
---

## 🧠 Concepto

La **Streams API** permite procesar colecciones de datos de forma declarativa y funcional. Un stream es una secuencia de elementos que se procesa en un **pipeline** de operaciones.

> [!important] Un stream NO almacena datos — los procesa.

## Crear un Stream

```java
// Desde colección
List<String> nombres = List.of("Ana", "Juber", "Carlos");
Stream<String> stream = nombres.stream();

// Stream paralelo
Stream<String> paralelo = nombres.parallelStream();

// Desde valores
Stream<Integer> numeros = Stream.of(1, 2, 3, 4, 5);

// Stream infinito (usar con limit!)
Stream<Integer> infinito = Stream.iterate(0, n -> n + 1);
Stream<Double> randoms = Stream.generate(Math::random);

// IntStream / LongStream / DoubleStream (primitivos)
IntStream rango = IntStream.range(1, 11);  // 1 a 10
IntStream rangoCerrado = IntStream.rangeClosed(1, 10); // 1 a 10
```

## Operaciones Intermedias (devuelven Stream)

```java
List<String> nombres = List.of("Ana", "Juber", "Carlos", "Ana", "pedro");

// filter — filtrar por condición
nombres.stream()
    .filter(n -> n.length() > 3)  // ["Juber", "Carlos", "pedro"]

// map — transformar cada elemento
nombres.stream()
    .map(String::toUpperCase)  // ["ANA", "JUBER", "CARLOS", "ANA", "PEDRO"]

// flatMap — aplanar streams de streams
List<List<Integer>> listas = List.of(List.of(1,2), List.of(3,4));
listas.stream()
    .flatMap(Collection::stream)  // [1, 2, 3, 4]

// distinct — eliminar duplicados
nombres.stream()
    .distinct()  // ["Ana", "Juber", "Carlos", "pedro"]

// sorted — ordenar
nombres.stream()
    .sorted()  // orden natural
nombres.stream()
    .sorted(Comparator.comparingInt(String::length))  // por longitud

// limit / skip
nombres.stream().limit(3)  // primeros 3
nombres.stream().skip(2)   // omitir los 2 primeros

// peek — para debugging (no modifica el stream)
nombres.stream()
    .peek(n -> System.out.println("Procesando: " + n))
    .filter(n -> n.length() > 3)
```

## Operaciones Terminales (consumen el Stream)

```java
List<Integer> nums = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// forEach
nums.stream().forEach(System.out::println);

// collect — recolectar a colección
List<Integer> pares = nums.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// count
long cantidad = nums.stream().filter(n -> n > 5).count(); // 5

// findFirst / findAny
Optional<Integer> primero = nums.stream().filter(n -> n > 5).findFirst();

// anyMatch / allMatch / noneMatch
boolean hayMayores5 = nums.stream().anyMatch(n -> n > 5); // true
boolean todosMayores0 = nums.stream().allMatch(n -> n > 0); // true
boolean ningunNegativo = nums.stream().noneMatch(n -> n < 0); // true

// min / max
Optional<Integer> maximo = nums.stream().max(Comparator.naturalOrder());

// reduce — acumular
int suma = nums.stream().reduce(0, Integer::sum); // 55
Optional<Integer> producto = nums.stream().reduce((a, b) -> a * b);

// toArray
Object[] array = nums.stream().toArray();
Integer[] array2 = nums.stream().toArray(Integer[]::new);
```

## Collectors

```java
List<String> nombres = List.of("Ana", "Juber", "Carlos", "Ana");

// toList (Java 16+, inmutable)
List<String> lista = nombres.stream().collect(Collectors.toList());

// toSet
Set<String> conjunto = nombres.stream().collect(Collectors.toSet());

// joining — concatenar strings
String resultado = nombres.stream()
    .collect(Collectors.joining(", ", "[", "]")); // "[Ana, Juber, Carlos, Ana]"

// groupingBy
Map<Integer, List<String>> porLongitud = nombres.stream()
    .collect(Collectors.groupingBy(String::length));
// {3=[Ana, Ana], 5=[Juber], 6=[Carlos]}

// counting
Map<Integer, Long> conteo = nombres.stream()
    .collect(Collectors.groupingBy(String::length, Collectors.counting()));

// partitioningBy
Map<Boolean, List<String>> particion = nombres.stream()
    .collect(Collectors.partitioningBy(n -> n.length() > 3));

// summarizingInt
IntSummaryStatistics stats = nombres.stream()
    .collect(Collectors.summarizingInt(String::length));
// count, sum, min, average, max
```

## Pipeline completo — ejemplo real

```java
List<Producto> productos = obtenerProductos();

Map<String, Double> precioPromedioPorCategoria = productos.stream()
    .filter(p -> p.isDisponible())
    .collect(Collectors.groupingBy(
        Producto::getCategoria,
        Collectors.averagingDouble(Producto::getPrecio)
    ));
```

## Streams de primitivos

```java
// Suma, promedio, etc. directamente
int suma = IntStream.rangeClosed(1, 100).sum(); // 5050
OptionalDouble promedio = IntStream.of(1,2,3,4,5).average();

// Convertir a Stream de objetos
IntStream.range(1, 5)
    .mapToObj(i -> "Item " + i)
    .forEach(System.out::println);
```

## ⚠️ Errores Comunes

> [!warning] Un stream solo se puede consumir una vez
> ```java
> Stream<String> s = lista.stream();
> s.forEach(System.out::println);
> s.forEach(System.out::println); // ❌ IllegalStateException
> ```

> [!warning] `parallelStream()` no siempre es más rápido
> Solo conviene en colecciones grandes y operaciones costosas. En operaciones simples, el overhead de paralelismo lo hace más lento.

## 📌 Notas Rápidas
