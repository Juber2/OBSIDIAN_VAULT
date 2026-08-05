---
tags:
  - Java
  - Intermedio
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
  - Streams API
Palabras clave:
  - lambda, functional interface, Function, Predicate, Consumer, Supplier, method reference
---

## 🧠 Concepto

Las **lambdas** son funciones anónimas (sin nombre de clase ni método) que se pueden pasar como argumentos. Requieren una **Functional Interface** — interfaz con un único método abstracto.

## Sintaxis

```java
// Forma completa
(parametros) -> { cuerpo; return valor; }

// Expresión simple (sin llaves ni return)
(parametros) -> expresion

// Sin parámetros
() -> System.out.println("Hola")

// Un parámetro (sin paréntesis)
x -> x * 2

// Múltiples parámetros
(a, b) -> a + b
```

## Functional Interfaces del paquete `java.util.function`

### `Predicate<T>` — recibe T, devuelve boolean
```java
Predicate<String> esLargo = s -> s.length() > 5;
esLargo.test("Hola");     // false
esLargo.test("Hola mundo"); // true

// Composición
Predicate<Integer> mayor10 = n -> n > 10;
Predicate<Integer> par = n -> n % 2 == 0;
Predicate<Integer> mayorYPar = mayor10.and(par);
```

### `Function<T, R>` — recibe T, devuelve R
```java
Function<String, Integer> longitud = s -> s.length();
longitud.apply("Juber"); // 5

// Composición
Function<Integer, Integer> doble = n -> n * 2;
Function<Integer, Integer> masUno = n -> n + 1;
Function<Integer, Integer> dobleYmasUno = doble.andThen(masUno);
dobleYmasUno.apply(3); // 7
```

### `Consumer<T>` — recibe T, no devuelve nada
```java
Consumer<String> imprimir = s -> System.out.println(s);
imprimir.accept("Hola"); // imprime "Hola"
```

### `Supplier<T>` — no recibe nada, devuelve T
```java
Supplier<String> saludo = () -> "Hola mundo";
saludo.get(); // "Hola mundo"
```

### `BiFunction<T, U, R>` — recibe dos argumentos
```java
BiFunction<String, Integer, String> repetir = (s, n) -> s.repeat(n);
repetir.apply("Java", 3); // "JavaJavaJava"
```

### `UnaryOperator<T>` — recibe T, devuelve T
```java
UnaryOperator<String> mayusculas = s -> s.toUpperCase();
```

### `BinaryOperator<T>` — recibe dos T, devuelve T
```java
BinaryOperator<Integer> sumar = (a, b) -> a + b;
```

## Crear tu propia Functional Interface

```java
@FunctionalInterface
public interface Calculadora {
    int operar(int a, int b);
}

// Uso
Calculadora suma = (a, b) -> a + b;
Calculadora resta = (a, b) -> a - b;
suma.operar(5, 3); // 8
```

## Method References (referencias a métodos)

Forma más concisa de lambdas cuando solo llaman a un método existente.

```java
// Lambda vs Method Reference
lista.forEach(s -> System.out.println(s));
lista.forEach(System.out::println); // equivalente

// Referencia a método estático
Function<String, Integer> parse = Integer::parseInt;

// Referencia a método de instancia de un tipo
Function<String, String> upper = String::toUpperCase;

// Referencia a método de instancia de un objeto específico
String prefijo = "Hola ";
Function<String, String> saludar = prefijo::concat;

// Referencia a constructor
Supplier<ArrayList<String>> nuevaLista = ArrayList::new;
```

## Tabla resumen

| Interface | Signatura | Método |
|-----------|-----------|--------|
| `Predicate<T>` | `T → boolean` | `test()` |
| `Function<T,R>` | `T → R` | `apply()` |
| `Consumer<T>` | `T → void` | `accept()` |
| `Supplier<T>` | `() → T` | `get()` |
| `UnaryOperator<T>` | `T → T` | `apply()` |
| `BinaryOperator<T>` | `(T,T) → T` | `apply()` |

## ⚠️ Errores Comunes

> [!warning] Variables en lambdas deben ser effectively final
> Las variables locales usadas dentro de una lambda no pueden modificarse después.
> ```java
> int x = 5;
> Runnable r = () -> System.out.println(x); // OK — x no se modifica
> x = 10; // ❌ hace que x deje de ser effectively final
> ```

## 📌 Notas Rápidas
