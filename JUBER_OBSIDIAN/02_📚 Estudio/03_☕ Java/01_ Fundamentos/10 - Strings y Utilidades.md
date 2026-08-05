---
tags:
  - Java
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - Java Fundamentos
Palabras clave:
  - String, StringBuilder, métodos de String, Math, utilidades
---

## 🧠 String

En Java, `String` es inmutable — cada operación que "modifica" un String crea uno nuevo.

### Métodos útiles

```java
String texto = "  Hola Mundo  ";

texto.length();              // 14
texto.trim();                // "Hola Mundo" (sin espacios)
texto.toUpperCase();         // "  HOLA MUNDO  "
texto.toLowerCase();         // "  hola mundo  "
texto.charAt(2);             // 'H' (índice tras trim)
texto.contains("Mundo");     // true
texto.replace("Mundo", "Java"); // "  Hola Java  "
texto.startsWith("  Hola");  // true
texto.endsWith("do  ");      // true
texto.indexOf("Mundo");      // 6
texto.substring(2, 6);       // "Hola"
texto.split(" ");            // array de palabras
texto.isEmpty();             // false
texto.isBlank();             // false (Java 11+)
```

### Comparar Strings

```java
String a = "Hola";
String b = "hola";

a.equals(b);           // false (case sensitive)
a.equalsIgnoreCase(b); // true
```

### Concatenación eficiente — StringBuilder

```java
// Ineficiente (crea objetos nuevos en cada +)
String resultado = "";
for (int i = 0; i < 1000; i++) {
    resultado += i; // ❌
}

// Eficiente
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i); // ✅
}
String resultado = sb.toString();
```

### String.format / formatted

```java
String mensaje = String.format("Hola %s, tienes %d años", "Juber", 25);
// Java 15+
String mensaje2 = "Hola %s, tienes %d años".formatted("Juber", 25);
```

## 🧠 Clase Math

```java
Math.abs(-5);       // 5
Math.max(10, 20);   // 20
Math.min(10, 20);   // 10
Math.pow(2, 8);     // 256.0
Math.sqrt(16);      // 4.0
Math.round(4.6);    // 5
Math.random();      // número aleatorio entre 0.0 y 1.0
Math.PI;            // 3.141592653589793
```

## 🧠 Parseo de tipos

```java
int x = Integer.parseInt("42");
double d = Double.parseDouble("3.14");
String s = String.valueOf(100);  // "100"
```

## ⚠️ Errores Comunes

> [!warning] NumberFormatException
> `Integer.parseInt("abc")` lanza `NumberFormatException`. Siempre validar antes de parsear.

## 📌 Notas Rápidas
