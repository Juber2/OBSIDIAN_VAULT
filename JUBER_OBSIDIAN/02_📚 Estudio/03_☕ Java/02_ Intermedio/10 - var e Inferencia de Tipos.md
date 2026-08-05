---
tags:
  - Java
  - Intermedio
Creado: 2026-05-21
Relacionado:
  - Java Intermedio
Palabras clave:
  - var, inferencia de tipos, local variable type inference, Java 10
---

## 🧠 Concepto

`var` (Java 10+) permite que el compilador **infiera el tipo** de una variable local. No es tipado dinámico — el tipo se determina en compilación y no cambia.

```java
// Sin var
ArrayList<String> lista = new ArrayList<String>();
Map<String, List<Integer>> mapa = new HashMap<String, List<Integer>>();

// Con var — más legible
var lista = new ArrayList<String>();
var mapa = new HashMap<String, List<Integer>>();
```

## Dónde SÍ se puede usar

```java
// Variables locales
var nombre = "Juber";      // String
var edad = 25;             // int
var precio = 9.99;         // double
var lista = new ArrayList<String>();

// En bucles
for (var item : lista) {
    System.out.println(item);
}

for (var i = 0; i < 10; i++) {
    System.out.println(i);
}

// Try-with-resources
try (var reader = Files.newBufferedReader(Path.of("archivo.txt"))) {
    // reader es BufferedReader
}
```

## Dónde NO se puede usar

```java
// Campos de clase
private var nombre; // ❌

// Parámetros de métodos
public void metodo(var param) {} // ❌

// Tipo de retorno
public var obtenerNombre() {} // ❌

// Sin inicializar
var x; // ❌ — el compilador no puede inferir el tipo

// Con null sin tipo
var nulo = null; // ❌

// Lambdas
var lambda = () -> "Hola"; // ❌ (salvo en parámetros de lambda desde Java 11)
```

## var en lambdas (Java 11+)

```java
// Permite poner anotaciones en parámetros de lambda
(@NotNull var nombre, @NotNull var apellido) -> nombre + " " + apellido;
```

## Cuándo usar var

```java
// ✅ Cuando el tipo es obvio por el contexto
var usuario = new Usuario("Juber", "juber@email.com");
var nombres = new ArrayList<String>();

// ✅ Para reducir repetición de tipos largos
var entrada = new BufferedReader(new InputStreamReader(System.in));
var resultado = clienteHttp.enviar(request, HttpResponse.BodyHandlers.ofString());

// ❌ Cuando el tipo no es claro
var x = calcularAlgo(); // ¿qué devuelve?
var datos = obtener();   // ambiguo
```

## ⚠️ Errores Comunes

> [!warning] `var` no es tipado dinámico
> ```java
> var nombre = "Juber";
> nombre = 123; // ❌ — sigue siendo String, no puede cambiar de tipo
> ```

> [!warning] No abusar de var
> Usarlo donde mejora legibilidad, no como excusa para no escribir tipos. En APIs públicas siempre declarar el tipo explícitamente.

## 📌 Notas Rápidas
