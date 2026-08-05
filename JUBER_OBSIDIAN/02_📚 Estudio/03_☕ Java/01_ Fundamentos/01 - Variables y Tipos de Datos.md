---
tags:
  - Java
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - Java Fundamentos
Palabras clave:
  - variables, tipos de datos, primitivos, casting
---

## 🧠 Concepto

## Tipos Primitivos

| Tipo | Tamaño | Rango | Ejemplo |
|------|--------|-------|---------|
| `byte` | 8 bits | -128 a 127 | |
| `short` | 16 bits | -32,768 a 32,767 | |
| `int` | 32 bits | -2^31 a 2^31-1 | |
| `long` | 64 bits | -2^63 a 2^63-1 | |
| `float` | 32 bits | decimal simple precisión | |
| `double` | 64 bits | decimal doble precisión | |
| `char` | 16 bits | un carácter Unicode | |
| `boolean` | 1 bit | true / false | |

## Variables de Referencia

```java
String nombre = "Juber";
int[] numeros = {1, 2, 3};
```

## Declaración e inicialización

```java
// Declarar
int edad;

// Inicializar
edad = 25;

// Declarar e inicializar
int edad = 25;

// Constante (final)
final double PI = 3.1416;
```

## Casting

```java
// Implícito (widening) — sin pérdida de datos
int x = 10;
double y = x; // OK

// Explícito (narrowing) — puede perder datos
double a = 9.99;
int b = (int) a; // b = 9
```

## ⚠️ Errores Comunes

## 📌 Notas Rápidas
