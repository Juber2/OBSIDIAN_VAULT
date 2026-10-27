---
tags:
  - Java
  - Intermedio
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
Palabras clave:
  - generics, tipos parametrizados, wildcards, bounded types
---

## 🧠 Concepto

Los **Generics** permiten crear clases, interfaces y métodos que trabajan con cualquier tipo de dato, manteniendo la seguridad de tipos en tiempo de compilación. Evitan casting explícito y errores en runtime.

## Por qué existen

```java
// Sin generics — inseguro
List lista = new ArrayList();
lista.add("Hola");
lista.add(123); // mezcla tipos sin error
String texto = (String) lista.get(0); // casting manual, puede fallar

// Con generics — seguro
List<String> lista = new ArrayList<>();
lista.add("Hola");
lista.add(123); // ❌ error en compilación
String texto = lista.get(0); // sin casting
```

## Clases genéricas

```java
public class Caja<T> {
    private T contenido;

    public Caja(T contenido) {
        this.contenido = contenido;
    }

    public T getContenido() {
        return contenido;
    }
}

// Uso
Caja<String> cajaTexto = new Caja<>("Hola");
Caja<Integer> cajaNumero = new Caja<>(42);
```

## Múltiples parámetros de tipo

```java
public class Par<K, V> {
    private K clave;
    private V valor;

    public Par(K clave, V valor) {
        this.clave = clave;
        this.valor = valor;
    }
}

Par<String, Integer> par = new Par<>("edad", 25);
```

## Métodos genéricos

```java
public <T> void imprimir(T elemento) {
    System.out.println(elemento);
}

public <T extends Comparable<T>> T maximo(T a, T b) {
    return a.compareTo(b) > 0 ? a : b;
}
```

## Bounded Types — límites de tipo

```java
// Upper bound — T debe ser Number o subclase
public <T extends Number> double sumar(List<T> lista) {
    double total = 0;
    for (T elemento : lista) {
        total += elemento.doubleValue();
    }
    return total;
}

// Multiple bounds
public <T extends Comparable<T> & Cloneable> T metodo(T param) { ... }
```

## Wildcards (`?`)

```java
// Unbounded wildcard — cualquier tipo
public void imprimirLista(List<?> lista) {
    for (Object item : lista) {
        System.out.println(item);
    }
}

// Upper bounded wildcard — Number o subclases (lectura)
public double sumarNumeros(List<? extends Number> lista) {
    return lista.stream().mapToDouble(Number::doubleValue).sum();
}

// Lower bounded wildcard — Integer o superclases (escritura)
public void agregarEnteros(List<? super Integer> lista) {
    lista.add(1);
    lista.add(2);
}
```

## Regla PECS

> **Producer Extends, Consumer Super**
> - Si la lista **produce** datos que lees → `<? extends T>`
> - Si la lista **consume** datos que escribes → `<? super T>`

## Convenciones de nomenclatura

| Letra | Uso típico |
|-------|-----------|
| `T` | Type (tipo genérico) |
| `E` | Element (colecciones) |
| `K` | Key (mapa) |
| `V` | Value (mapa) |
| `N` | Number |
| `R` | Return type |

## Type Erasure

En tiempo de ejecución, Java elimina la información de tipos genéricos (**type erasure**). Todos los `List<String>`, `List<Integer>` etc. son simplemente `List` en runtime.

```java
List<String> strings = new ArrayList<>();
List<Integer> integers = new ArrayList<>();
System.out.println(strings.getClass() == integers.getClass()); // true
```

## ⚠️ Errores Comunes

> [!warning] No puedes crear arrays de tipos genéricos
> ```java
> T[] array = new T[10]; // ❌ no compila
> ```

> [!warning] No puedes usar primitivos directamente
> ```java
> List<int> lista; // ❌ — usar List<Integer>
> ```

## 📌 Notas Rápidas
- Generics existen solo en compile-time (type erasure) — en runtime, `List<String>` y `List<Integer>` son ambos `List`.
- `? extends T` (wildcard covariante) se usa para leer de una colección; `? super T` (contravariante) para escribir.
- Un método genérico puede tener su propio tipo, independiente de la clase: `<T> T primero(List<T> lista)`.
- Bounded types (`<T extends Comparable<T>>`) restringen qué tipos se aceptan, habilitando llamar métodos de esa cota dentro del genérico.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

