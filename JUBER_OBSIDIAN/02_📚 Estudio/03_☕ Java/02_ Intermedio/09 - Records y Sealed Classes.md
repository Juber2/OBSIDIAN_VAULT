---
tags:
  - Java
  - Intermedio
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
Palabras clave:
  - record, sealed, permits, Java 16, Java 17, inmutabilidad
---

## 🧠 Records (Java 16+)

Un `record` es una clase inmutable de datos con generación automática de constructor, getters, `equals()`, `hashCode()` y `toString()`.

### Sin record (verboso)
```java
public final class Punto {
    private final int x;
    private final int y;

    public Punto(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int x() { return x; }
    public int y() { return y; }

    @Override public boolean equals(Object o) { ... }
    @Override public int hashCode() { ... }
    @Override public String toString() { ... }
}
```

### Con record (conciso)
```java
public record Punto(int x, int y) {}

// Uso
Punto p = new Punto(3, 4);
p.x();         // 3
p.y();         // 4
p.toString();  // "Punto[x=3, y=4]"

Punto p2 = new Punto(3, 4);
p.equals(p2);  // true
```

### Personalizar un record

```java
public record Usuario(String nombre, String email) {

    // Constructor compacto — validaciones
    public Usuario {
        Objects.requireNonNull(nombre, "nombre no puede ser null");
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Email inválido");
        }
        nombre = nombre.trim(); // puedes transformar los valores
    }

    // Métodos adicionales
    public String dominio() {
        return email.substring(email.indexOf("@") + 1);
    }

    // static fields y métodos
    public static Usuario crear(String nombre, String email) {
        return new Usuario(nombre.toLowerCase(), email.toLowerCase());
    }
}
```

### Records e interfaces

```java
public interface Describible {
    String descripcion();
}

public record Producto(String nombre, double precio) implements Describible {
    @Override
    public String descripcion() {
        return nombre + " - $" + precio;
    }
}
```

### Cuándo usar Records

- DTOs (Data Transfer Objects)
- Value Objects en DDD
- Respuestas de API
- Claves compuestas

```java
// DTO de respuesta
public record UsuarioResponse(Long id, String nombre, String email) {}

// Value object
public record Money(BigDecimal amount, String currency) {}
```

---

## 🧠 Sealed Classes (Java 17+)

Una `sealed class` o `sealed interface` restringe qué clases pueden extenderla o implementarla.

```java
// Solo estas clases pueden extender Figura
public sealed class Figura
    permits Circulo, Rectangulo, Triangulo {}

public final class Circulo extends Figura {
    private final double radio;
    public Circulo(double radio) { this.radio = radio; }
}

public final class Rectangulo extends Figura {
    private final double ancho, alto;
    public Rectangulo(double ancho, double alto) {
        this.ancho = ancho;
        this.alto = alto;
    }
}

public non-sealed class Triangulo extends Figura {
    // non-sealed: permite extensión libre
}
```

### Sealed + Pattern Matching (Java 21)

```java
double area = switch (figura) {
    case Circulo c -> Math.PI * c.radio() * c.radio();
    case Rectangulo r -> r.ancho() * r.alto();
    case Triangulo t -> calcularAreaTriangulo(t);
    // No necesita default porque el compilador sabe todas las subclases
};
```

### Palabras clave

| Keyword | Significado |
|---------|-------------|
| `sealed` | Clase/interfaz que restringe subclases |
| `permits` | Lista de clases permitidas |
| `final` | No puede ser extendida más |
| `non-sealed` | Puede ser extendida libremente |

## ⚠️ Errores Comunes

> [!warning] Los records son inmutables por diseño
> No puedes agregar campos de instancia no-finales. Si necesitas mutabilidad, usa una clase normal.

> [!warning] Los records no pueden heredar de otra clase
> Implícitamente extienden `java.lang.Record`. Sí pueden implementar interfaces.

## 📌 Notas Rápidas
- Un `record` genera automáticamente constructor, getters (sin prefijo `get`), `equals()`, `hashCode()` y `toString()` — ideal para DTOs.
- Se puede agregar validación en un "compact constructor" sin repetir la lista de parámetros:
  ```java
  public record Rango(int min, int max) {
      public Rango {
          if (min > max) throw new IllegalArgumentException("min > max");
      }
  }
  ```
- `sealed` restringe qué clases pueden extender/implementar una clase o interface (`permits`), habilitando pattern matching exhaustivo en `switch` sin necesitar `default`.
- Records + sealed classes + pattern matching en `switch` (Java 21) es la base para modelar dominios cerrados de forma segura, similar a data classes + sealed classes en Kotlin.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

