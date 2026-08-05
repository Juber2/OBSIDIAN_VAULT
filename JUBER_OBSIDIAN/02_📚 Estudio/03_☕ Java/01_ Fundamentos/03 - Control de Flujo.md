---
tags:
  - Java
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - Java Fundamentos
Palabras clave:
  - if, switch, for, while, do-while, bucles
---

## 🧠 Concepto

## if / else if / else

```java
if (condicion) {
    // ...
} else if (otraCondicion) {
    // ...
} else {
    // ...
}
```

## switch

```java
// Clásico
switch (dia) {
    case 1:
        System.out.println("Lunes");
        break;
    case 2:
        System.out.println("Martes");
        break;
    default:
        System.out.println("Otro día");
}

// Moderno (Java 14+)
String nombre = switch (dia) {
    case 1 -> "Lunes";
    case 2 -> "Martes";
    default -> "Otro día";
};
```

## for

```java
// Clásico
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}

// Enhanced for (for-each)
int[] numeros = {1, 2, 3, 4, 5};
for (int n : numeros) {
    System.out.println(n);
}
```

## while

```java
int i = 0;
while (i < 5) {
    System.out.println(i);
    i++;
}
```

## do-while

```java
// Se ejecuta al menos una vez
int i = 0;
do {
    System.out.println(i);
    i++;
} while (i < 5);
```

## break y continue

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) break;    // sale del bucle
    if (i % 2 == 0) continue; // salta esta iteración
    System.out.println(i);
}
```

## ⚠️ Errores Comunes

## 📌 Notas Rápidas
