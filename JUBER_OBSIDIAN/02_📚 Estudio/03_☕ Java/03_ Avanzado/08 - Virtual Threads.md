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
  - virtual threads, Project Loom, Java 21, lightweight threads, structured concurrency
---

## 🧠 Concepto

Los **Virtual Threads** (Java 21, Project Loom) son hilos ultra-ligeros gestionados por la JVM, no por el SO. Permiten escribir código concurrente bloqueante con la misma simplicidad que el código secuencial, pero con la escala de programación asíncrona.

## El problema que resuelven

```
Hilo de plataforma (Platform Thread):
- Mapea 1:1 con hilo del SO
- Costoso: ~1MB de stack
- Un servidor típico → máximo ~1000 hilos concurrentes

Virtual Thread:
- Gestionado por la JVM (M:N mapping)
- Ultra-ligero: ~1KB
- Un servidor puede tener millones de virtual threads
```

## Crear Virtual Threads

```java
// Crear y arrancar un virtual thread
Thread vThread = Thread.ofVirtual().start(() -> {
    System.out.println("Virtual thread: " + Thread.currentThread());
});

// Con nombre
Thread vThread = Thread.ofVirtual()
    .name("mi-vthread")
    .start(() -> procesarPeticion());

// ExecutorService con virtual threads (recomendado con Spring Boot 3.2+)
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        executor.submit(() -> procesarPeticion());
    }
} // auto-close espera que terminen
```

## Comparativa con platform threads

```java
// Platform threads — pool fijo, costoso
ExecutorService platform = Executors.newFixedThreadPool(200);

// Virtual threads — uno por tarea, escala a millones
ExecutorService virtual = Executors.newVirtualThreadPerTaskExecutor();

// Las dos APIs son idénticas — la diferencia es interna
```

## Cómo funcionan internamente

```
Virtual Thread (bloqueado en I/O)
        ↓
JVM detecta bloqueo → desmonta del carrier thread
        ↓
Carrier thread queda libre para otro virtual thread
        ↓
Cuando I/O termina → virtual thread se remonta en un carrier thread disponible
```

Los **carrier threads** son platform threads (del pool ForkJoinPool) que "montan" virtual threads.

## Cuándo usar Virtual Threads

```java
// ✅ Ideal para — workloads I/O bound
// - peticiones HTTP
// - consultas a base de datos
// - lectura/escritura de archivos
// - llamadas a servicios externos

// ❌ No aporta beneficio — workloads CPU bound
// - cálculos matemáticos intensivos
// - procesamiento de imágenes
// Para esto: ForkJoinPool o parallelStream
```

## Habilitarlo en Spring Boot 3.2+

```properties
# application.properties
spring.threads.virtual.enabled=true
```

```java
// O programáticamente
@Bean
public TomcatProtocolHandlerCustomizer<?> customizeProtocolHandler() {
    return protocolHandler ->
        protocolHandler.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
}
```

## Structured Concurrency (Java 21 Preview)

Nueva API para manejar múltiples tareas concurrentes como una unidad:

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<Usuario> usuario = scope.fork(() -> buscarUsuario(id));
    Future<List<Pedido>> pedidos = scope.fork(() -> buscarPedidos(id));

    scope.join();           // esperar ambas
    scope.throwIfFailed();  // si alguna falló, lanzar excepción

    return new UsuarioConPedidos(usuario.get(), pedidos.get());
}
```

## ⚠️ Consideraciones

> [!warning] Pinning — evitar `synchronized` con I/O en virtual threads
> Los bloques `synchronized` pueden "pinear" el virtual thread al carrier thread, anulando el beneficio. Usar `ReentrantLock` en su lugar cuando sea crítico.

> [!warning] Thread-locals con virtual threads
> Si usas `ThreadLocal` intensivamente con millones de virtual threads, el consumo de memoria puede ser alto. Considera `ScopedValue` (Java 21 Preview).

## 📌 Notas Rápidas
- No usar un pool para Virtual Threads (`newVirtualThreadPerTaskExecutor()` crea uno nuevo por tarea, no reutiliza) — el modelo mental es distinto al de platform threads, donde sí importa limitar el pool.
- Habilitar `spring.threads.virtual.enabled=true` en Spring Boot 3.2+ cambia el executor de Tomcat, sin tocar el código de los controllers — es prácticamente "gratis" para apps IO-bound.
- No resuelven cuellos de botella de CPU ni de la base de datos — si el límite real es el pool de conexiones a la BD o el CPU, Virtual Threads no lo arregla por sí solo.
- Es la respuesta directa de Java al problema que motivó los frameworks reactivos (WebFlux, Mutiny en Quarkus) — con Virtual Threads se puede escribir código bloqueante simple y aun así escalar a alta concurrencia.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

