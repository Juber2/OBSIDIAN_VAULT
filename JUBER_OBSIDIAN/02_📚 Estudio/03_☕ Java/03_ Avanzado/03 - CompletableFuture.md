---
tags:
  - Java
  - Avanzado
  - Concurrencia
Creado: 2026-05-21
Relacionado:
  - Java Avanzado
  - ExecutorService y Thread Pools
Palabras clave:
  - CompletableFuture, async, thenApply, thenCompose, allOf, anyOf, programación asíncrona
---

## 🧠 Concepto

`CompletableFuture` (Java 8+) permite programación asíncrona no bloqueante con un API fluido para encadenar operaciones, combinar resultados y manejar errores.

## Crear CompletableFuture

```java
// Tarea asíncrona sin retorno
CompletableFuture<Void> cf = CompletableFuture.runAsync(() -> {
    System.out.println("Ejecutando en hilo: " + Thread.currentThread().getName());
});

// Tarea asíncrona con retorno
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
    return "Resultado";
});

// Con executor personalizado
ExecutorService executor = Executors.newFixedThreadPool(4);
CompletableFuture<String> cf = CompletableFuture.supplyAsync(
    () -> "Resultado", executor
);

// Ya completado
CompletableFuture<String> completo = CompletableFuture.completedFuture("Ya listo");
```

## Encadenar operaciones (pipeline)

```java
CompletableFuture<String> resultado = CompletableFuture
    .supplyAsync(() -> "juber")           // produce String
    .thenApply(String::toUpperCase)        // transforma: String → String
    .thenApply(s -> "Hola, " + s);        // transforma: String → String

resultado.get(); // "Hola, JUBER"
```

### thenApply vs thenCompose

```java
// thenApply — mapper devuelve un valor directo (Function<T, R>)
.thenApply(nombre -> nombre.toUpperCase())

// thenCompose — mapper devuelve CompletableFuture (evita anidar futuros)
.thenCompose(nombre -> buscarUsuarioPorNombre(nombre))
// donde buscarUsuarioPorNombre devuelve CompletableFuture<Usuario>
```

### thenAccept y thenRun

```java
// thenAccept — consume el resultado sin transformar
cf.thenAccept(resultado -> System.out.println("Resultado: " + resultado));

// thenRun — ejecuta acción sin acceder al resultado
cf.thenRun(() -> System.out.println("Completado"));
```

## Combinar futuros

```java
CompletableFuture<String> futuro1 = CompletableFuture.supplyAsync(() -> "Hola");
CompletableFuture<String> futuro2 = CompletableFuture.supplyAsync(() -> " Mundo");

// thenCombine — combinar dos resultados
CompletableFuture<String> combinado = futuro1.thenCombine(futuro2, (a, b) -> a + b);
combinado.get(); // "Hola Mundo"

// allOf — esperar que TODOS terminen
CompletableFuture<Void> todos = CompletableFuture.allOf(futuro1, futuro2);
todos.join(); // espera ambos

// anyOf — esperar que CUALQUIERA termine
CompletableFuture<Object> cualquiera = CompletableFuture.anyOf(futuro1, futuro2);
```

## Manejo de errores

```java
CompletableFuture<String> cf = CompletableFuture
    .supplyAsync(() -> {
        if (Math.random() > 0.5) throw new RuntimeException("Fallo");
        return "OK";
    })
    .exceptionally(ex -> {
        System.out.println("Error: " + ex.getMessage());
        return "Valor por defecto";
    });

// handle — maneja tanto éxito como error
.handle((resultado, ex) -> {
    if (ex != null) return "Error: " + ex.getMessage();
    return "Éxito: " + resultado;
});

// whenComplete — ejecutar siempre (sin transformar)
.whenComplete((resultado, ex) -> {
    if (ex != null) System.out.println("Error: " + ex.getMessage());
    else System.out.println("OK: " + resultado);
});
```

## Ejemplo real — llamadas en paralelo

```java
// Llamar dos servicios en paralelo y combinar resultados
CompletableFuture<Usuario> usuarioCF = CompletableFuture
    .supplyAsync(() -> usuarioService.findById(id));

CompletableFuture<List<Pedido>> pedidosCF = CompletableFuture
    .supplyAsync(() -> pedidoService.findByUsuario(id));

CompletableFuture<UsuarioConPedidos> resultado = usuarioCF.thenCombine(
    pedidosCF,
    (usuario, pedidos) -> new UsuarioConPedidos(usuario, pedidos)
);

resultado.get(); // ambas llamadas se hicieron en paralelo
```

## get() vs join()

```java
cf.get();   // lanza checked exceptions (InterruptedException, ExecutionException)
cf.join();  // lanza unchecked CompletionException — más cómodo en streams
```

## ⚠️ Errores Comunes

> [!warning] Bloquear con `get()` en hilos del pool
> Si llamas `get()` dentro de otro `CompletableFuture`, puedes agotar el pool de hilos. Usar `thenCompose` en su lugar.

> [!warning] No manejar excepciones
> Sin `exceptionally` o `handle`, una excepción en la cadena se silencia hasta que llamas `get()`.

## 📌 Notas Rápidas
- `thenApply` transforma el resultado (síncrono); `thenCompose` encadena otro `CompletableFuture` (evita futuros anidados, equivalente a `flatMap`).
- `thenCombine` une los resultados de dos `CompletableFuture` independientes que corrieron en paralelo.
- Sin especificar un executor, las callbacks corren en el `ForkJoinPool.commonPool()` compartido por toda la JVM — para trabajo pesado, pasar un `Executor` propio (`thenApplyAsync(fn, executor)`).
- Es la base conceptual de la programación reactiva en Java (y de `Uni`/`Multi` en Quarkus, ver [[04 - REST con RESTEasy Reactive]]) — encadenar sin bloquear en vez de esperar resultados secuencialmente.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

