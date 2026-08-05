---
tags:
  - Java
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - Java Fundamentos
Palabras clave:
  - operadores, aritméticos, lógicos, comparación
---

## 🧠 Concepto

## Aritméticos

| Operador | Descripción | Ejemplo |
|----------|-------------|---------|
| `+` | Suma | `5 + 3 = 8` |
| `-` | Resta | `5 - 3 = 2` |
| `*` | Multiplicación | `5 * 3 = 15` |
| `/` | División | `10 / 3 = 3` (entero) |
| `%` | Módulo (resto) | `10 % 3 = 1` |
| `++` | Incremento | `i++` |
| `--` | Decremento | `i--` |

## Relacionales / Comparación

| Operador | Descripción |
|----------|-------------|
| `==` | Igual a |
| `!=` | Diferente de |
| `>` | Mayor que |
| `<` | Menor que |
| `>=` | Mayor o igual |
| `<=` | Menor o igual |

## Lógicos

| Operador | Descripción |
|----------|-------------|
| `&&` | AND (ambos true) |
| `\|\|` | OR (al menos uno true) |
| `!` | NOT (negación) |

## Asignación

```java
int x = 10;
x += 5;  // x = 15
x -= 3;  // x = 12
x *= 2;  // x = 24
x /= 4;  // x = 6
x %= 4;  // x = 2
```

## Ternario

```java
// condicion ? valor_si_true : valor_si_false
String resultado = (edad >= 18) ? "Mayor de edad" : "Menor de edad";
```

## ⚠️ Errores Comunes

> [!warning] `==` vs `.equals()`
> Para Strings y objetos, `==` compara referencias (dirección en memoria), no el valor.
> Usar siempre `.equals()` para comparar contenido.
> ```java
> String a = "Hola";
> String b = "Hola";
> a == b       // puede ser false (depende del contexto)
> a.equals(b)  // siempre true si el contenido es igual
> ```

## 📌 Notas Rápidas
