---
tags:
  - Java
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - Java Fundamentos
Palabras clave:
  - métodos, funciones, parámetros, retorno, sobrecarga
---

## 🧠 Concepto

Un método es un bloque de código reutilizable que realiza una tarea específica.

## Estructura

```java
[modificador] [tipoRetorno] nombreMetodo([parámetros]) {
    // cuerpo
    return valor; // si no es void
}
```

## Ejemplos

```java
// Sin retorno, sin parámetros
public void saludar() {
    System.out.println("Hola");
}

// Con retorno
public int sumar(int a, int b) {
    return a + b;
}

// Con retorno y múltiples parámetros
public String formatearNombre(String nombre, String apellido) {
    return nombre + " " + apellido;
}
```

## Modificadores de acceso

| Modificador | Accesible desde |
|-------------|-----------------|
| `public` | Cualquier clase |
| `private` | Solo la misma clase |
| `protected` | Misma clase + subclases + mismo paquete |
| (sin modificador) | Solo el mismo paquete |

## static

```java
// Se puede llamar sin instanciar la clase
public static int multiplicar(int a, int b) {
    return a * b;
}

// Llamada:
int resultado = MiClase.multiplicar(3, 4);
```

## Sobrecarga (Overloading)

```java
// Mismo nombre, distintos parámetros
public int sumar(int a, int b) { return a + b; }
public double sumar(double a, double b) { return a + b; }
public int sumar(int a, int b, int c) { return a + b + c; }
```

## ⚠️ Errores Comunes

## 📌 Notas Rápidas
