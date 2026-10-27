---
tags:
  - Java
  - Intermedio
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
  - Streams API
Palabras clave:
  - Optional, null, NullPointerException, orElse, map, flatMap
---

## 🧠 Concepto

`Optional<T>` es un contenedor que puede o no tener un valor. Su objetivo es evitar `NullPointerException` y hacer explícito que un valor puede estar ausente.

## Crear Optional

```java
// Con valor
Optional<String> conValor = Optional.of("Hola");

// Puede ser null (usar cuando el valor puede ser null)
Optional<String> nullable = Optional.ofNullable(null);

// Vacío explícitamente
Optional<String> vacio = Optional.empty();
```

> [!warning] `Optional.of(null)` lanza NullPointerException. Usa `Optional.ofNullable()` cuando el valor puede ser null.

## Verificar y obtener el valor

```java
Optional<String> opt = Optional.of("Juber");

opt.isPresent(); // true
opt.isEmpty();   // false (Java 11+)

opt.get(); // "Juber" — lanza NoSuchElementException si está vacío (evitar sin verificar)

// Seguro
if (opt.isPresent()) {
    String valor = opt.get();
}
```

## Métodos útiles

```java
Optional<String> opt = Optional.ofNullable(obtenerNombre());

// orElse — valor por defecto si está vacío
String nombre = opt.orElse("Sin nombre");

// orElseGet — valor por defecto con Supplier (lazy)
String nombre = opt.orElseGet(() -> generarNombreDefecto());

// orElseThrow — lanzar excepción si está vacío
String nombre = opt.orElseThrow(() -> new RuntimeException("No se encontró nombre"));

// ifPresent — ejecutar acción si hay valor
opt.ifPresent(n -> System.out.println("Nombre: " + n));

// ifPresentOrElse (Java 9+)
opt.ifPresentOrElse(
    n -> System.out.println("Nombre: " + n),
    () -> System.out.println("Sin nombre")
);

// map — transformar el valor si existe
Optional<Integer> longitud = opt.map(String::length);

// flatMap — cuando el mapper devuelve Optional
Optional<String> ciudad = opt.flatMap(n -> buscarCiudad(n));

// filter — filtrar por condición
Optional<String> largo = opt.filter(n -> n.length() > 3);
```

## Patrón de uso en capas

```java
// Repository
public Optional<Usuario> findById(Long id) {
    return Optional.ofNullable(usuarios.get(id));
}

// Service
public String obtenerNombreUsuario(Long id) {
    return usuarioRepository.findById(id)
        .map(Usuario::getNombre)
        .orElseThrow(() -> new UsuarioNotFoundException(id));
}
```

## Cuándo usar Optional

| ✅ Usar | ❌ No usar |
|--------|----------|
| Retorno de métodos que pueden no encontrar resultado | Parámetros de métodos |
| Evitar null explícito en return | Campos de clases / entidades |
| APIs de servicio/repositorio | Colecciones (usar colección vacía en su lugar) |

## ⚠️ Errores Comunes

> [!warning] Encadenar `.get()` sin verificar
> ```java
> String nombre = optional.get(); // ❌ puede explotar si está vacío
> String nombre = optional.orElse("default"); // ✅
> ```

> [!warning] Usar Optional como campo de clase
> ```java
> public class Usuario {
>     private Optional<String> email; // ❌ mala práctica
>     private String email; // ✅ usar null o String vacío
> }
> ```

## 📌 Notas Rápidas
- `Optional` está pensado como **tipo de retorno**, no como tipo de parámetro ni de campo de clase.
- `orElseGet(() -> ...)` difiere el cálculo del default (lazy); `orElse(valor)` siempre lo evalúa, incluso si no hace falta.
- `orElseThrow()` sin argumento lanza `NoSuchElementException`; con argumento (`orElseThrow(MiExcepcion::new)`) lanza la excepción indicada.
- `map()` sobre un Optional vacío simplemente no ejecuta la función y devuelve `Optional.empty()` — encadenable sin checks manuales de null.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

