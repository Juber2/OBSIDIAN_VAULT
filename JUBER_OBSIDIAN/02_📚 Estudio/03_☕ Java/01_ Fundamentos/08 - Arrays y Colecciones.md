---
tags:
  - Java
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - Java Fundamentos
Palabras clave:
  - arrays, ArrayList, HashMap, List, Map, colecciones
---

## 🧠 Arrays

Tamaño fijo, tipo homogéneo.

```java
// Declarar e inicializar
int[] numeros = new int[5];
String[] nombres = {"Ana", "Juber", "Carlos"};

// Acceder
System.out.println(nombres[0]); // Ana

// Recorrer
for (String nombre : nombres) {
    System.out.println(nombre);
}

// Longitud
System.out.println(nombres.length); // 3
```

## 🧠 Colecciones (java.util)

### ArrayList — lista dinámica

```java
import java.util.ArrayList;

ArrayList<String> lista = new ArrayList<>();
lista.add("Java");
lista.add("Python");
lista.remove("Python");
lista.get(0);        // "Java"
lista.size();        // 1
lista.contains("Java"); // true

// Recorrer
for (String item : lista) {
    System.out.println(item);
}
```

### HashMap — clave/valor

```java
import java.util.HashMap;

HashMap<String, Integer> edades = new HashMap<>();
edades.put("Juber", 25);
edades.put("Ana", 22);
edades.get("Juber");       // 25
edades.containsKey("Ana"); // true
edades.remove("Ana");

// Recorrer
for (String clave : edades.keySet()) {
    System.out.println(clave + ": " + edades.get(clave));
}
```

### HashSet — sin duplicados

```java
import java.util.HashSet;

HashSet<String> conjunto = new HashSet<>();
conjunto.add("Java");
conjunto.add("Java"); // ignorado, ya existe
conjunto.size(); // 1
```

## Comparativa rápida

| Estructura | Ordenado | Duplicados | Clave/Valor | Uso típico |
|------------|----------|------------|-------------|------------|
| `Array` | ✅ índice | ✅ | ❌ | Tamaño fijo conocido |
| `ArrayList` | ✅ índice | ✅ | ❌ | Lista dinámica general |
| `HashMap` | ❌ | claves únicas | ✅ | Búsqueda por clave |
| `HashSet` | ❌ | ❌ | ❌ | Unicidad de elementos |

## ⚠️ Errores Comunes

> [!warning] ArrayIndexOutOfBoundsException
> Acceder a un índice que no existe en el array.
> ```java
> int[] arr = {1, 2, 3};
> arr[5]; // ❌ ArrayIndexOutOfBoundsException
> ```

## 📌 Notas Rápidas
- Los arrays tienen tamaño fijo al crearse; las colecciones (`List`, `Set`, `Map`) crecen dinámicamente.
- `ArrayList` es rápido para acceso por índice pero lento para insertar/eliminar al inicio; `LinkedList` es al revés.
- `HashMap` no garantiza orden; `LinkedHashMap` mantiene orden de inserción; `TreeMap` mantiene orden natural/comparador.
- Iterar y modificar una colección al mismo tiempo con un `for-each` lanza `ConcurrentModificationException` — usar `Iterator.remove()` o `removeIf()`.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

