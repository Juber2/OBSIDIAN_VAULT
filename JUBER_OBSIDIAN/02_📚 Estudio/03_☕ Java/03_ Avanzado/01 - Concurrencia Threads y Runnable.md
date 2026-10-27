---
tags:
  - Java
  - Avanzado
  - Concurrencia
Creado: 2026-05-21
Relacionado:
  - Java Avanzado
Palabras clave:
  - Thread, Runnable, synchronized, volatile, race condition, deadlock
---

## 🧠 Concepto

La **concurrencia** permite ejecutar múltiples tareas de forma simultánea. En Java, la unidad básica es el **Thread** (hilo de ejecución).

## Crear un Thread

```java
// Opción 1 — extender Thread
public class MiThread extends Thread {
    @Override
    public void run() {
        System.out.println("Ejecutando en: " + Thread.currentThread().getName());
    }
}
new MiThread().start();

// Opción 2 — implementar Runnable (preferida)
Runnable tarea = () -> System.out.println("Hilo: " + Thread.currentThread().getName());
Thread hilo = new Thread(tarea, "mi-hilo");
hilo.start();

// Opción 3 — lambda directa
new Thread(() -> System.out.println("Lambda thread")).start();
```

> [!warning] Llamar `run()` en vez de `start()` no crea un nuevo hilo — ejecuta en el hilo actual.

## Estado de un Thread

```
NEW → RUNNABLE → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED
```

```java
Thread t = new Thread(() -> { ... });
t.getState(); // Thread.State.NEW
t.start();
t.getState(); // Thread.State.RUNNABLE
t.join();     // esperar a que termine
t.getState(); // Thread.State.TERMINATED
```

## Métodos importantes

```java
Thread.currentThread().getName(); // nombre del hilo actual
Thread.sleep(1000);               // pausar 1 segundo (lanza InterruptedException)
hilo.join();                      // esperar que el hilo termine
hilo.interrupt();                 // interrumpir el hilo
hilo.isAlive();                   // ¿está corriendo?
hilo.setDaemon(true);             // hilo daemon (termina con la JVM)
```

## Problemas de concurrencia

### Race Condition — condición de carrera
```java
// PROBLEMA: dos hilos modifican la misma variable
public class Contador {
    private int cuenta = 0;

    public void incrementar() {
        cuenta++; // NO es atómica: read → modify → write
    }
}
// Con 1000 hilos sumando, el resultado puede no ser 1000
```

### synchronized — sincronización
```java
public class ContadorSeguro {
    private int cuenta = 0;

    // Sincronizar el método completo
    public synchronized void incrementar() {
        cuenta++;
    }

    // Sincronizar un bloque específico
    public void incrementarBloque() {
        synchronized (this) {
            cuenta++;
        }
    }
}
```

### volatile — visibilidad entre hilos
```java
public class Configuracion {
    private volatile boolean activo = true; // cambios visibles a todos los hilos

    public void detener() { activo = false; }
    public boolean estaActivo() { return activo; }
}
```

> [!note] `volatile` garantiza visibilidad pero NO atomicidad. Para operaciones compuestas (read-modify-write), usar `synchronized` o clases `Atomic`.

## Clases Atomic (java.util.concurrent.atomic)

```java
AtomicInteger contador = new AtomicInteger(0);
contador.incrementAndGet();  // thread-safe
contador.addAndGet(5);
contador.compareAndSet(5, 10); // si valor==5, cambia a 10
```

## Deadlock

Dos hilos esperando mutuamente el recurso del otro.

```java
// Hilo 1 tiene lock A, quiere lock B
// Hilo 2 tiene lock B, quiere lock A → DEADLOCK

// Prevención: adquirir locks siempre en el mismo orden
```

## ⚠️ Errores Comunes

> [!warning] No compartir estado mutable entre hilos sin sincronización
> Usar objetos inmutables, `synchronized`, clases `Atomic` o estructuras thread-safe.

> [!warning] `Thread.sleep()` no libera el lock
> Si un hilo está `sleep()` dentro de un bloque `synchronized`, otros hilos siguen bloqueados.

## 📌 Notas Rápidas
- Implementar `Runnable` es preferible a extender `Thread` — permite que la clase herede de otra cosa, y separa "qué se ejecuta" de "cómo se ejecuta".
- `volatile` garantiza visibilidad entre hilos (evita que uno vea un valor cacheado viejo), pero no atomicidad — para contadores concurrentes usar `AtomicInteger`/`AtomicLong`.
- `synchronized` garantiza tanto exclusión mutua como visibilidad, pero tiene mayor costo que alternativas lock-free cuando hay alta contención.
- Antes de multithreading manual, evaluar si `CompletableFuture`, `ExecutorService` o (Java 21+) Virtual Threads resuelven el problema con menos código propenso a errores.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

