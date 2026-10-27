---
tags:
  - Java
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - Java Fundamentos
Palabras clave:
  - excepciones, try, catch, finally, throw, throws, Exception
---

## 🧠 Concepto

Las excepciones son eventos que interrumpen el flujo normal del programa. Java las maneja con `try/catch`.

## try / catch / finally

```java
try {
    int resultado = 10 / 0; // ArithmeticException
} catch (ArithmeticException e) {
    System.out.println("Error: " + e.getMessage());
} finally {
    // Siempre se ejecuta, haya o no excepción
    System.out.println("Finalizado");
}
```

## Múltiples catch

```java
try {
    String texto = null;
    texto.length(); // NullPointerException

    int[] arr = new int[3];
    arr[10] = 1; // ArrayIndexOutOfBoundsException

} catch (NullPointerException e) {
    System.out.println("Referencia nula: " + e.getMessage());
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Índice fuera de rango: " + e.getMessage());
} catch (Exception e) {
    // Captura cualquier otra excepción (más genérico, va al final)
    System.out.println("Error general: " + e.getMessage());
}
```

## throw — lanzar excepción

```java
public void validarEdad(int edad) {
    if (edad < 0) {
        throw new IllegalArgumentException("La edad no puede ser negativa");
    }
}
```

## throws — declarar que un método puede lanzar excepción

```java
public void leerArchivo(String ruta) throws IOException {
    // código que puede lanzar IOException
}
```

## Excepciones comunes

| Excepción | Causa típica |
|-----------|-------------|
| `NullPointerException` | Usar objeto no inicializado |
| `ArrayIndexOutOfBoundsException` | Índice fuera del array |
| `ArithmeticException` | División entre cero |
| `NumberFormatException` | Parsear texto no numérico |
| `IllegalArgumentException` | Argumento inválido |
| `ClassCastException` | Cast inválido entre tipos |
| `IOException` | Error de entrada/salida |

## Excepciones personalizadas

```java
public class SaldoInsuficienteException extends RuntimeException {
    public SaldoInsuficienteException(String mensaje) {
        super(mensaje);
    }
}

// Uso
throw new SaldoInsuficienteException("Saldo insuficiente para realizar la operación");
```

## ⚠️ Errores Comunes

> [!warning] Capturar Exception genérica primero
> El catch más específico debe ir primero. Si pones `catch (Exception e)` antes que los específicos, nunca llegarás a ellos.

## 📌 Notas Rápidas
- Checked exceptions (`IOException`) se deben declarar o manejar obligatoriamente; unchecked (`RuntimeException` y sus hijas) no.
- `try-with-resources` cierra automáticamente cualquier `AutoCloseable`, incluso si hay una excepción — preferible a `finally` manual.
- Crear excepciones propias extendiendo `RuntimeException` (no checked) es la práctica más común en backends modernos, para no forzar `throws` en cada firma.
- Nunca capturar `Exception` o `Throwable` genérico y "tragarte" el error sin loguearlo — dificulta muchísimo el debugging en producción.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

