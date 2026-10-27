---
tags:
  - Java
  - Avanzado
  - JVM
Creado: 2026-05-21
Relacionado:
  - Java Avanzado
Palabras clave:
  - JVM, heap, stack, garbage collection, GC, metaspace, OutOfMemoryError
---

## 🧠 Arquitectura de la JVM

```
JVM
├── Class Loader Subsystem     → carga los .class
├── Runtime Data Areas         → memoria
│   ├── Heap                   → objetos, GC actúa aquí
│   ├── Stack                  → frames de métodos, variables locales
│   ├── Metaspace              → metadata de clases (Java 8+, reemplaza PermGen)
│   ├── PC Register            → contador de instrucción por hilo
│   └── Native Method Stack    → métodos nativos (C/C++)
└── Execution Engine           → interpreta/compila bytecode
    ├── Interpreter
    ├── JIT Compiler
    └── Garbage Collector
```

## Heap — donde viven los objetos

```
Heap
├── Young Generation (Eden + Survivor S0 + Survivor S1)
│   └── Minor GC — frecuente, rápido
└── Old Generation (Tenured)
    └── Major GC / Full GC — costoso, pausas largas
```

### Ciclo de vida de un objeto en el Heap

1. Objeto creado → **Eden**
2. Sobrevive Minor GC → **Survivor**
3. Sobrevive varios GC → **Old Generation**
4. Ya no tiene referencias → **elegible para GC**

## Stack — variables locales y llamadas a métodos

- Cada hilo tiene su propio Stack
- Cada llamada a método crea un **frame** con variables locales y parámetros
- Al retornar, el frame se elimina automáticamente
- `StackOverflowError` → recursión infinita o stack demasiado profundo

## Garbage Collection

El GC libera memoria automáticamente cuando un objeto no tiene referencias alcanzables.

### Tipos de GC en Java

| GC | Uso | Características |
|----|-----|-----------------|
| **Serial GC** | Aplicaciones pequeñas, un core | `-XX:+UseSerialGC` |
| **Parallel GC** | Throughput alto, múltiples cores | default en Java 8 |
| **G1 GC** | Balance latencia/throughput | default desde Java 9 |
| **ZGC** | Latencia ultra baja (<10ms) | Java 15+ estable |
| **Shenandoah** | Similar a ZGC | GraalVM/OpenJDK |

### Forzar GC (no recomendado)
```java
System.gc(); // sugiere al JVM que ejecute GC, no garantizado
```

## Flags JVM importantes

```bash
# Tamaño del heap
-Xms512m          # tamaño inicial
-Xmx2g            # tamaño máximo

# Ver GC en consola
-XX:+PrintGCDetails
-Xlog:gc*          # Java 9+

# Tipo de GC
-XX:+UseG1GC
-XX:+UseZGC

# Tamaño del stack por hilo
-Xss512k
```

## Errores de memoria comunes

```java
// OutOfMemoryError: Java heap space
// — heap lleno, objeto muy grande, memory leak
List<byte[]> lista = new ArrayList<>();
while (true) lista.add(new byte[1024 * 1024]); // llena el heap

// OutOfMemoryError: Metaspace
// — demasiadas clases cargadas dinámicamente

// StackOverflowError
void recursionInfinita() { recursionInfinita(); } // stack overflow
```

## Memory Leaks en Java

Aunque Java tiene GC, pueden ocurrir memory leaks si objetos siguen siendo referenciados sin intención.

```java
// Ejemplo clásico: static collections que crecen sin límite
public class Cache {
    private static final Map<String, Object> CACHE = new HashMap<>();

    public static void agregar(String key, Object valor) {
        CACHE.put(key, valor); // nunca se limpia → memory leak
    }
}

// Solución: usar WeakHashMap, Caffeine, Guava Cache, o limpiar explícitamente
```

## Herramientas de monitoreo

| Herramienta | Uso |
|-------------|-----|
| `jvisualvm` | GUI para monitoreo de heap, threads, GC |
| `jconsole` | Monitoreo básico JMX |
| `jstat` | Estadísticas de GC desde consola |
| `jmap` | Dump del heap |
| `jstack` | Dump de threads (detectar deadlocks) |
| IntelliJ Profiler | Profiling integrado |
| Async Profiler | Profiler de bajo overhead |

## ⚠️ Errores Comunes

> [!warning] `finalize()` está deprecado
> No usar `finalize()` para liberar recursos. Usar `try-with-resources` o `Cleaner`.

> [!tip] Regla de thumb para heap
> Para aplicaciones web con Spring Boot: `-Xms256m -Xmx512m` como punto de partida, ajustar según profiling real.

## 📌 Notas Rápidas
- El heap se divide en generaciones (Young/Eden + Survivor, Old/Tenured) — la mayoría de los objetos mueren jóvenes, por eso el GC generacional es eficiente (recolecta Young con más frecuencia y más barato).
- G1GC es el recolector default desde Java 9 — buen balance para la mayoría de apps; ZGC/Shenandoah son opciones de baja latencia para heaps muy grandes.
- Un `OutOfMemoryError` no siempre es "falta de RAM" — puede ser un memory leak (referencias que nunca se liberan, ej. listeners no removidos, caches sin límite).
- `-Xms` fija el heap inicial, `-Xmx` el máximo — igualarlos evita que la JVM tenga que redimensionar el heap en caliente, a costa de reservar más memoria desde el arranque.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

