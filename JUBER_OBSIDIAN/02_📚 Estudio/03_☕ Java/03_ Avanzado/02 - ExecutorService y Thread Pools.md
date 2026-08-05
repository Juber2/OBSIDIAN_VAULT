---
tags:
  - Java
  - Avanzado
  - Concurrencia
Creado: 2026-05-21
Relacionado:
  - Java Avanzado
  - Concurrencia Threads y Runnable
Palabras clave:
  - ExecutorService, ThreadPool, Callable, Future, ScheduledExecutorService
---

## 🧠 Concepto

Crear un `Thread` por cada tarea es costoso. El **ExecutorService** gestiona un pool de hilos reutilizables, mejorando el rendimiento y el control.

## Crear ExecutorService

```java
// Pool fijo — N hilos siempre activos
ExecutorService executor = Executors.newFixedThreadPool(4);

// Pool caché — crea hilos según demanda, los reutiliza
ExecutorService executor = Executors.newCachedThreadPool();

// Un solo hilo — secuencial
ExecutorService executor = Executors.newSingleThreadExecutor();

// Virtual Thread executor (Java 21) — muy recomendado
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
```

## Ejecutar tareas

```java
ExecutorService executor = Executors.newFixedThreadPool(4);

// submit Runnable — sin retorno
executor.submit(() -> System.out.println("Tarea 1"));

// submit Callable — con retorno
Future<String> futuro = executor.submit(() -> {
    Thread.sleep(1000);
    return "Resultado";
});

// Obtener resultado (bloqueante)
String resultado = futuro.get();
String resultado = futuro.get(5, TimeUnit.SECONDS); // con timeout

futuro.isDone();    // ¿terminó?
futuro.cancel(true); // cancelar

// Cerrar el executor (SIEMPRE hacerlo)
executor.shutdown(); // espera que terminen las tareas pendientes
executor.shutdownNow(); // interrumpe tareas en curso
```

## invokeAll e invokeAny

```java
List<Callable<String>> tareas = List.of(
    () -> "Resultado 1",
    () -> "Resultado 2",
    () -> "Resultado 3"
);

// invokeAll — esperar todas
List<Future<String>> futuros = executor.invokeAll(tareas);
for (Future<String> f : futuros) {
    System.out.println(f.get());
}

// invokeAny — retorna el primero que termina
String primero = executor.invokeAny(tareas);
```

## ScheduledExecutorService

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

// Ejecutar una vez después de un delay
scheduler.schedule(() -> System.out.println("Después de 5s"), 5, TimeUnit.SECONDS);

// Ejecutar periódicamente (con fixed rate — cuenta desde el inicio)
scheduler.scheduleAtFixedRate(
    () -> System.out.println("Cada 2s"),
    0,    // delay inicial
    2,    // período
    TimeUnit.SECONDS
);

// Ejecutar periódicamente (con fixed delay — cuenta desde que termina)
scheduler.scheduleWithFixedDelay(
    () -> System.out.println("2s después de terminar"),
    0, 2, TimeUnit.SECONDS
);
```

## ThreadPoolExecutor personalizado

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2,                          // corePoolSize
    10,                         // maximumPoolSize
    60L,                        // keepAliveTime
    TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(100), // cola de tareas
    new ThreadPoolExecutor.CallerRunsPolicy() // política si la cola está llena
);
```

## Patrón correcto de uso

```java
ExecutorService executor = Executors.newFixedThreadPool(
    Runtime.getRuntime().availableProcessors() // núcleos disponibles
);

try {
    List<Future<String>> futuros = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        final int id = i;
        futuros.add(executor.submit(() -> procesarTarea(id)));
    }
    for (Future<String> f : futuros) {
        System.out.println(f.get());
    }
} finally {
    executor.shutdown();
}
```

## ⚠️ Errores Comunes

> [!danger] No cerrar el ExecutorService
> La JVM no termina si hay hilos activos en el pool. Siempre llamar `shutdown()` en un bloque `finally`.

> [!warning] `future.get()` puede lanzar `ExecutionException`
> Si la tarea lanzó una excepción, `get()` la envuelve en `ExecutionException`. Siempre manejarla.

## 📌 Notas Rápidas
