---
tags:
  - Java
  - Avanzado
  - Concurrencia
Creado: 2026-09-08
Relacionado:
  - Java Avanzado
  - 08 - Virtual Threads
Palabras clave:
  - StructuredTaskScope, concurrencia estructurada, cancelacion, alcance, hilos virtuales
---

## 🧠 Concepto

**Concurrencia estructurada** es la idea de que las tareas concurrentes deben tener el mismo ciclo de vida que el bloque de código que las lanza: si el bloque termina, todas sus tareas terminaron o fueron canceladas.

Es la misma disciplina que trajo la programación estructurada a los saltos: donde antes había `goto`, ahora hay bloques con entrada y salida únicas. Con `ExecutorService`, las tareas concurrentes son el `goto` de la concurrencia.

> [!warning] Estado de la API
> `StructuredTaskScope` viene en **vista previa** desde Java 21 y su API cambió entre versiones (JEP 453, 462, 480, 499, 505). Los ejemplos de acá usan la forma más difundida; **verificá la firma exacta de tu JDK** antes de escribir código.
> El concepto es estable; la sintaxis todavía no.

## ❌ El problema

```java
// ExecutorService: las tareas escapan del alcance
public Perfil obtenerPerfil(UUID id) throws Exception {
    var pool = Executors.newFixedThreadPool(2);

    Future<Usuario> usuario = pool.submit(() -> usuarioService.buscar(id));
    Future<Saldo>   saldo   = pool.submit(() -> cuentasService.saldo(id));

    return new Perfil(usuario.get(), saldo.get());
    // ⚠️ si usuario.get() lanza excepción, la tarea del saldo SIGUE CORRIENDO
    // ⚠️ nadie cierra el pool
}
```

Los problemas concretos:

| Problema | Consecuencia |
|---|---|
| Una falla y la otra sigue | trabajo inútil, recursos consumidos |
| Si el hilo padre se interrumpe | las tareas hijas siguen huérfanas |
| Fugas de hilos | pools sin cerrar |
| Trazas rotas | el stacktrace no relaciona padre con hijo |
| `get()` bloqueante en orden fijo | esperás la primera aunque la segunda ya falló |

## ✅ La solución

```java
public Perfil obtenerPerfil(UUID id) throws Exception {
    try (var alcance = new StructuredTaskScope.ShutdownOnFailure()) {

        var usuario = alcance.fork(() -> usuarioService.buscar(id));
        var saldo   = alcance.fork(() -> cuentasService.saldo(id));

        alcance.join();                 // espera a todas
        alcance.throwIfFailed();        // propaga la primera excepción

        return new Perfil(usuario.get(), saldo.get());
    }
    // Al salir del try: TODAS las tareas terminaron o fueron canceladas ✅
}
```

> [!tip] El `try-with-resources` es la garantía
> Si una tarea falla, las demás se cancelan automáticamente. Si el hilo padre se interrumpe, las hijas también. Al salir del bloque, **no queda nada corriendo**. Esa es toda la idea.

## 🎯 Las dos políticas

### `ShutdownOnFailure` — necesito todos los resultados

```java
try (var alcance = new StructuredTaskScope.ShutdownOnFailure()) {
    var cliente     = alcance.fork(() -> clienteService.buscar(id));
    var cuentas     = alcance.fork(() -> cuentaService.deCliente(id));
    var preferencias = alcance.fork(() -> prefService.de(id));

    alcance.join();
    alcance.throwIfFailed(ErrorDePerfilException::new);

    return new Perfil(cliente.get(), cuentas.get(), preferencias.get());
}
```

Si cualquiera falla, las demás se cancelan de inmediato: no se espera a que terminen tareas cuyo resultado ya no sirve.

### `ShutdownOnSuccess` — me alcanza con la primera

```java
// Consultar tres proveedores de tipo de cambio: gana el más rápido
try (var alcance = new StructuredTaskScope.ShutdownOnSuccess<TipoDeCambio>()) {
    alcance.fork(() -> proveedorA.consultar(par));
    alcance.fork(() -> proveedorB.consultar(par));
    alcance.fork(() -> proveedorC.consultar(par));

    alcance.join();
    return alcance.result();      // el primero que respondió; los otros cancelados
}
```

> [!tip] El patrón de "petición cubierta" (*hedged request*)
> Lanzar la misma consulta a varias réplicas y quedarse con la primera respuesta reduce el p99 de forma notable: si una réplica está lenta, otra responde. Con concurrencia estructurada son cuatro líneas y sin fugas.

## ⏱️ Plazos

```java
try (var alcance = new StructuredTaskScope.ShutdownOnFailure()) {
    var evaluacion = alcance.fork(() -> antifraude.evaluar(operacion));
    var limites    = alcance.fork(() -> limitesService.verificar(operacion));

    alcance.joinUntil(Instant.now().plusMillis(100));    // presupuesto estricto
    alcance.throwIfFailed();

    return decidir(evaluacion.get(), limites.get());

} catch (TimeoutException e) {
    return Decision.degradada();      // las tareas ya fueron canceladas ✅
}
```

> [!tip] El plazo se aplica al grupo, no a cada tarea
> Con `Future` habría que calcular el tiempo restante para cada `get()` a mano. Acá el presupuesto es del bloque entero, que es como se piensa en realidad — es exactamente lo que necesita el motor de [[C8 - Limites y Antifraude]].

## 🧵 Con hilos virtuales

```java
// Cada fork crea un HILO VIRTUAL por defecto: crear miles no es problema
try (var alcance = new StructuredTaskScope.ShutdownOnFailure()) {
    var futuros = cuentas.stream()
        .map(c -> alcance.fork(() -> saldoService.obtener(c)))    // 10.000 forks: OK
        .toList();

    alcance.join();
    alcance.throwIfFailed();

    return futuros.stream().map(Supplier::get).toList();
}
```

> [!warning] Miles de hilos virtuales sí; miles de conexiones a la base, no
> Los hilos virtuales son baratos, pero cada tarea que consulte la base toma una conexión del pool. Lanzar 10.000 forks contra un pool de 10 conexiones no acelera nada: genera 10.000 hilos esperando.
> **El cuello de botella se traslada, no desaparece.** Si la operación toca un recurso limitado, hay que limitar la concurrencia con un semáforo — ver [[08 - Virtual Threads]].

```java
var semaforo = new Semaphore(20);      // como máximo 20 consultas simultáneas
alcance.fork(() -> {
    semaforo.acquire();
    try { return saldoService.obtener(cuenta); }
    finally { semaforo.release(); }
});
```

## 🌳 Anidamiento

```java
public Reporte generar(UUID clienteId) throws Exception {
    try (var externo = new StructuredTaskScope.ShutdownOnFailure()) {

        var cuentas = externo.fork(() -> cuentaService.deCliente(clienteId));
        var perfil  = externo.fork(() -> {

            // Un alcance anidado: sus tareas son "nietas"
            try (var interno = new StructuredTaskScope.ShutdownOnFailure()) {
                var datos   = interno.fork(() -> clienteService.datos(clienteId));
                var scoring = interno.fork(() -> riesgoService.puntaje(clienteId));
                interno.join();
                interno.throwIfFailed();
                return new Perfil(datos.get(), scoring.get());
            }
        });

        externo.join();
        externo.throwIfFailed();
        return new Reporte(cuentas.get(), perfil.get());
    }
}
```

> [!tip] El árbol de tareas es real y observable
> A diferencia de un `ExecutorService`, la relación padre-hijo existe en el runtime: se ve en los volcados de hilos, en el depurador y en las herramientas de diagnóstico. Cancelar el padre cancela todo el subárbol.
> Eso convierte una traza de concurrencia —históricamente ilegible— en algo que se puede seguir.

## 📊 Comparación

| | `ExecutorService` | `CompletableFuture` | Concurrencia estructurada |
|---|---|---|---|
| Ciclo de vida acotado | ❌ | ❌ | ✅ |
| Cancelación en cascada | manual | manual | ✅ automática |
| Manejo de errores | por `Future` | encadenado, difícil | centralizado |
| Legibilidad | media | baja con muchos pasos | **alta** |
| Composición asíncrona | limitada | ✅ excelente | limitada |
| Estado de la API | estable | estable | **vista previa** |

> [!tip] No reemplaza a `CompletableFuture`
> Resuelven cosas distintas. `CompletableFuture` sirve para **encadenar** transformaciones asíncronas (`thenApply`, `thenCompose`). La concurrencia estructurada sirve para **lanzar tareas en paralelo y esperarlas** dentro de un bloque, en estilo bloqueante.
> Para un backend que hace varias llamadas y espera todas —el caso más frecuente—, la estructurada es mucho más legible.

## 🧪 Tests

```java
@Test
void si_una_tarea_falla_las_demas_se_cancelan() throws Exception {
    var otraCancelada = new AtomicBoolean(false);

    assertThrows(ExecutionException.class, () -> {
        try (var alcance = new StructuredTaskScope.ShutdownOnFailure()) {
            alcance.fork(() -> { throw new RuntimeException("falla"); });
            alcance.fork(() -> {
                try { Thread.sleep(5000); }
                catch (InterruptedException e) { otraCancelada.set(true); throw e; }
                return "nunca";
            });
            alcance.join();
            alcance.throwIfFailed();
        }
    });

    assertTrue(otraCancelada.get());       // ✅ se canceló sola
}

@Test
void al_salir_del_bloque_no_queda_ninguna_tarea_corriendo() { ... }

@Test
void el_plazo_se_aplica_al_grupo_completo() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que anulan la garantía
> - **Olvidar `join()`**: `get()` sobre una tarea sin unir lanza `IllegalStateException`.
> - **`fork` fuera del hilo dueño del alcance**: solo el hilo que lo creó puede lanzar tareas.
> - **Guardar la referencia del alcance** para usarla después del bloque: ya está cerrado.
> - **Miles de forks contra un recurso limitado**: el cuello se traslada al pool de conexiones.
> - **Usarlo para encadenar transformaciones**: para eso está `CompletableFuture`.
> - **Escribir código de producción con una API en vista previa** sin asumir que va a cambiar.

## 📌 Notas Rápidas
- La frase que resume el valor: **si el bloque terminó, no quedó nada corriendo**. Es una garantía que `ExecutorService` nunca dio y que elimina una familia entera de fugas.
- Va de la mano con [[08 - Virtual Threads]]: los hilos virtuales hacen barato crear miles de tareas, y la concurrencia estructurada hace que ese poder no se convierta en caos.
- Al estar en vista previa, hay que compilar con `--enable-preview`. Para producción conviene esperar a que se estabilice, pero **conocerlo es valioso**: es hacia donde va la concurrencia en Java.
- En una entrevista, explicar por qué `ExecutorService` deja tareas huérfanas y cómo la concurrencia estructurada lo resuelve demuestra que seguís la evolución del lenguaje.

## 🔗 Relaciones
- **Módulo**: [[Java Avanzado]]
- **Anterior**: [[11 - Internals de Colecciones]]
- **Siguiente**: [[13 - Pattern Matching Moderno]]
- **Se conecta con**: [[08 - Virtual Threads]], [[03 - CompletableFuture]], [[02 - ExecutorService y Thread Pools]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

