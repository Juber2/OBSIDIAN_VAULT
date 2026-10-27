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

> [!warning] Sobrecarga ambigua
> Si dos métodos sobrecargados pueden aplicar igual de bien a una llamada, el compilador falla en vez de "adivinar".
> ```java
> void metodo(int a, long b) {}
> void metodo(long a, int b) {}
> metodo(1, 2); // ❌ ambiguo, no compila
> ```

> [!warning] Java pasa por valor, incluso para objetos
> Se pasa la copia de la **referencia**, no el objeto. Podés mutar el objeto apuntado, pero reasignar el parámetro dentro del método no afecta la variable original.
> ```java
> void reasignar(StringBuilder sb) {
>     sb = new StringBuilder("nuevo"); // no afecta al caller
> }
> void mutar(StringBuilder sb) {
>     sb.append("!"); // SÍ afecta al caller (mismo objeto)
> }
> ```

## 📌 Notas Rápidas
- La sobrecarga (overloading) se resuelve en compile-time; el override (herencia) se resuelve en runtime.
- Un método `static` no puede acceder a atributos/métodos de instancia sin una referencia explícita al objeto.
- Los parámetros varargs (`int... nums`) se tratan como un array dentro del método.
- Un método puede tener como máximo un parámetro varargs, y debe ser el último de la lista.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

