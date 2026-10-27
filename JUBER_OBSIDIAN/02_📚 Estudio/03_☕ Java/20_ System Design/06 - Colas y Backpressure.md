---
tags:
  - Java
  - System Design
  - Resiliencia
Creado: 2026-09-08
Relacionado:
  - System Design
  - 07 - Mensajeria Gestionada
Palabras clave:
  - backpressure, ley de Little, cola acotada, descarte de carga, prioridad, lotes
---

## 🧠 Concepto

Una cola desacopla a quien produce trabajo de quien lo consume. Absorbe picos, tolera consumidores caídos y permite escalarlos por separado.

Y trae un problema propio: **si el productor es más rápido que el consumidor, la cola crece sin límite** — hasta que algo se rompe.

```
Productor: 1.000 msg/s   →   [ cola ]   →   Consumidor: 500 msg/s

Crece 500 msg/s. En una hora: 1,8 millones de mensajes acumulados.
```

## 📐 La ley de Little

La herramienta de dimensionamiento más útil, y una fórmula de una línea:

```
L = λ × W

L = elementos en el sistema
λ = tasa de llegada (por segundo)
W = tiempo que cada uno pasa dentro
```

```
Ejemplo — dimensionar un pool de hilos:
  λ = 100 peticiones/s
  W = 200 ms = 0,2 s
  L = 100 × 0,2 = 20 peticiones simultáneas

  → hacen falta ~20 hilos. Con 10, la cola crece; con 200, desperdiciás memoria.
```

```
Ejemplo — dimensionar consumidores:
  Llegan 1.000 mensajes/s, cada uno tarda 50 ms
  L = 1.000 × 0,05 = 50 procesamientos simultáneos
  → 50 hilos consumidores, o 10 instancias con 5 hilos cada una
```

> [!tip] Sirve también al revés
> Si la cola tiene 10.000 mensajes y se procesan 500/s, el retraso es `10.000 / 500 = 20 segundos`. Ese número es el que responde "¿cuánto tarda en drenarse?" durante un incidente.

## 🚧 Backpressure

Es la señal de "vas muy rápido, frená". Sin ella, la presión se acumula hasta que algo revienta.

| Estrategia | Qué hace | Cuándo |
|---|---|---|
| **Bloquear** | el productor espera | productor propio y tolerante |
| **Descartar** | se tiran mensajes | telemetría, métricas |
| **Rechazar** | error explícito al productor | APIs (`429` / `503`) |
| **Almacenar en disco** | buffer persistente | picos previsibles |
| **Escalar consumidores** | más capacidad | la solución de fondo |

```java
// ❌ Cola sin límite: el productor nunca frena y la memoria se agota
var ejecutor = new ThreadPoolExecutor(10, 10, 0L, MILLISECONDS,
    new LinkedBlockingQueue<>());              // ¡ilimitada!

// ✅ Cola acotada + política de rechazo
var ejecutor = new ThreadPoolExecutor(10, 20, 60L, SECONDS,
    new LinkedBlockingQueue<>(1000),           // ← límite explícito
    new ThreadPoolExecutor.CallerRunsPolicy());  // el productor ejecuta: se frena solo
```

> [!danger] `Executors.newFixedThreadPool()` usa una cola ilimitada
> Es la fábrica que todo el mundo usa y esconde una `LinkedBlockingQueue` sin límite. Bajo carga sostenida, la cola crece hasta consumir el heap y la aplicación muere con `OutOfMemoryError` — sin ninguna señal previa de que había un problema.
> **Construí el `ThreadPoolExecutor` a mano con una cola acotada.**

| Política de rechazo | Comportamiento |
|---|---|
| `AbortPolicy` (default) | lanza `RejectedExecutionException` |
| **`CallerRunsPolicy`** | lo ejecuta el hilo que envió: **backpressure natural** |
| `DiscardPolicy` | lo descarta en silencio ⚠️ |
| `DiscardOldestPolicy` | descarta el más viejo de la cola |

> [!tip] `CallerRunsPolicy` implementa backpressure sin código
> Cuando la cola se llena, el hilo productor ejecuta la tarea él mismo. Mientras lo hace, no produce más. **El sistema se autorregula**: la tasa de entrada baja automáticamente a la de procesamiento.

## 🛡️ Descarte de carga

Cuando el sistema está saturado, rechazar rápido es mejor que aceptar todo y morir lentamente.

```java
@Component
public class ControlDeCarga {

    private final AtomicInteger enCurso = new AtomicInteger();
    private static final int LIMITE = 100;

    public <T> T ejecutar(Supplier<T> operacion) {
        if (enCurso.incrementAndGet() > LIMITE) {
            enCurso.decrementAndGet();
            throw new SistemaSobrecargadoException();      // → 503 + Retry-After
        }
        try { return operacion.get(); }
        finally { enCurso.decrementAndGet(); }
    }
}
```

> [!danger] Aceptar todo bajo saturación es peor que rechazar
> Si aceptás mil peticiones cuando podés con cien, **las mil fallan por timeout**: el usuario espera 30 segundos y recibe un error igual, y mientras tanto consumiste recursos por trabajo que nadie va a leer.
> Rechazando novecientas de inmediato, cien se completan bien. Es la diferencia entre degradación y colapso.

```
Sin descarte de carga:      1.000 peticiones → 1.000 timeouts (0% de éxito)
Con descarte de carga:      1.000 peticiones →   100 exitosas + 900 rechazos rápidos
```

> [!tip] Descartá en el borde, no en el fondo
> Rechazar en el gateway cuesta casi nada. Rechazar después de haber consultado la base y tomado un bloqueo desperdició todo ese trabajo. **Cuanto antes se rechace, mejor.**

## 🎯 Prioridades

```java
record Tarea(int prioridad, Runnable trabajo) implements Comparable<Tarea> {
    public int compareTo(Tarea otra) { return Integer.compare(otra.prioridad, prioridad); }
}

var ejecutor = new ThreadPoolExecutor(10, 10, 0L, MILLISECONDS,
    new PriorityBlockingQueue<>(1000));
```

```
Prioridad 10 · transferencias de clientes    ← nunca se descartan
Prioridad  5 · notificaciones
Prioridad  1 · reportes y analítica          ← los primeros en descartarse
```

> [!warning] `PriorityBlockingQueue` no es acotada
> Es otra trampa de la biblioteca estándar: acepta un tamaño inicial pero **crece sin límite**. Si usás prioridades, el control de tamaño lo tenés que implementar vos.

> [!danger] Cuidado con la inanición
> Si siempre hay tareas de prioridad alta, las de prioridad baja nunca se ejecutan. Los reportes quedan en la cola para siempre. La solución habitual es **colas separadas con capacidad reservada** para cada nivel, en vez de una sola cola con prioridades.

## 📦 Procesamiento por lotes

```java
// ❌ Una operación por mensaje
@KafkaListener(topics = "movimientos")
public void alRecibir(MovimientoEvento evento) {
    repo.save(convertir(evento));            // 1 INSERT por mensaje
}

// ✅ En lotes
@KafkaListener(topics = "movimientos", batch = "true")
public void alRecibirLote(List<MovimientoEvento> eventos, Acknowledgment ack) {
    repo.saveAll(eventos.stream().map(this::convertir).toList());   // 1 INSERT masivo
    ack.acknowledge();
}
```

| | Uno por uno | En lotes de 500 |
|---|---|---|
| Viajes a la base | 500 | 1 |
| Rendimiento | referencia | 10-50× |
| Latencia individual | menor | mayor (espera al lote) |
| Granularidad del fallo | por mensaje | todo el lote |

> [!tip] El lote intercambia latencia por rendimiento
> Para procesamiento asíncrono es casi siempre la elección correcta: nadie está esperando. Para una petición síncrona, no.
> El punto delicado es el manejo de errores: si un mensaje del lote falla, ¿se descarta todo? Lo habitual es reintentar el lote y, si vuelve a fallar, procesarlo de a uno para aislar el culpable.

## 📈 Escalar consumidores

```yaml
# Escalar por profundidad de cola, no por CPU
--scale-rule-name cola
--scale-rule-type azure-queue
--scale-rule-metadata queueName=transferencias queueLength=100
--min-replicas 2
--max-replicas 50
```

> [!tip] Para consumidores, la métrica correcta es la profundidad de la cola
> La CPU de un consumidor que espera E/S se mantiene baja mientras la cola crece: escalar por CPU no reacciona nunca. La longitud de la cola es la señal directa de que falta capacidad.

> [!danger] El límite real puede estar aguas abajo
> Escalar a 50 consumidores que escriben en una base que admite 100 conexiones no ayuda: el cuello se traslada. Antes de subir el máximo de réplicas, verificá que la dependencia aguante — ver [[06 - Bases de Datos Gestionadas]].

## 📊 Qué monitorear

| Métrica | Alerta |
|---|---|
| **Profundidad de la cola** | crece sostenidamente |
| **Antigüedad del mensaje más viejo** | > SLA de procesamiento |
| Tasa de entrada vs de salida | entrada > salida por N minutos |
| Tareas rechazadas | > 0 sostenido |
| Profundidad de la DLQ | > 0 |
| Tiempo de procesamiento (p99) | creciente |

> [!tip] La antigüedad del mensaje más viejo es mejor señal que la profundidad
> Una cola con 10.000 mensajes que se drena en 10 segundos está bien. Una con 100 mensajes donde el más viejo lleva 2 horas está rota — probablemente un mensaje envenenado bloqueando la partición.

```java
@Scheduled(fixedRate = 30_000)
public void medir() {
    metricas.gauge("cola.profundidad", cliente.contarMensajes(cola));
    metricas.gauge("cola.antiguedad.segundos", antiguedadDelMasViejo().getSeconds());
    metricas.gauge("cola.dlq.profundidad", cliente.contarMensajes(cola + "-dlq"));
}
```

## 🎛️ Diseñar el flujo

```
¿La respuesta la necesita el usuario ahora?
├── SÍ → síncrono, con timeout y descarte de carga
└── NO → cola
         ├── ¿el orden importa? → clave de partición / sesión
         ├── ¿puede duplicarse? → idempotencia (siempre)
         ├── ¿hay prioridades?  → colas separadas por nivel
         └── ¿el pico es previsible? → dimensionar el buffer
```

> [!tip] Lo que puede esperar, que espere
> Una transferencia debe confirmarse al usuario en el momento. El correo de notificación, el evento de antifraude, la actualización de la proyección y el registro analítico pueden ir a una cola.
> Mover trabajo fuera del camino crítico es lo que baja el p99 sin optimizar una sola línea de código.

## 🧪 Tests

```java
@Test
void la_cola_acotada_rechaza_cuando_se_llena() {
    llenarCola();
    assertThrows(RejectedExecutionException.class, () -> ejecutor.submit(tarea));
}

@Test
void con_CallerRunsPolicy_el_productor_se_frena() {
    long inicio = System.nanoTime();
    IntStream.range(0, 10_000).forEach(i -> ejecutor.submit(tareaLenta));
    // el productor tardó porque ejecutó parte del trabajo
    assertTrue(Duration.ofNanos(System.nanoTime() - inicio).toSeconds() > 1);
}

@Test
void bajo_saturacion_se_rechaza_rapido_en_vez_de_acumular() {
    var resultados = enviarEnParalelo(1000);
    assertTrue(resultados.rechazados() > 0);
    assertTrue(resultados.latenciaMediaDeRechazo().toMillis() < 50);   // rechazo rápido
}

@Test
void el_procesamiento_en_lote_reduce_las_consultas() {
    procesarLote(500);
    assertTrue(contadorDeConsultas.total() < 10);
}

@Test
void un_mensaje_envenenado_no_bloquea_la_cola() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que producen colapsos
> - **Colas sin límite**: `OutOfMemoryError` sin aviso previo.
> - **`Executors.newFixedThreadPool()`**: cola ilimitada escondida.
> - **`PriorityBlockingQueue`** creyendo que está acotada.
> - **Aceptar todo bajo saturación**: todo falla por timeout.
> - **Descartar en el fondo** en vez de en el borde.
> - **Escalar consumidores por CPU** en vez de por profundidad de cola.
> - **Escalar sin verificar la dependencia**: el cuello se traslada.
> - **Sin métrica de antigüedad**: la profundidad sola engaña.
> - **Prioridades sin capacidad reservada**: inanición de las tareas bajas.
> - **Trabajo diferible en el camino crítico**: p99 alto sin motivo.

## 📌 Notas Rápidas
- La ley de Little (`L = λ × W`) es la fórmula más rentable de esta nota: dimensiona pools de hilos, cantidad de consumidores y tiempo de drenado, todo con una multiplicación.
- `CallerRunsPolicy` es la forma más elegante de tener backpressure en Java: el sistema se autorregula sin lógica adicional.
- La cola ilimitada de `Executors.newFixedThreadPool()` es de las trampas más comunes de la biblioteca estándar. En una entrevista, mencionarla demuestra que leíste el código, no solo la documentación.
- Se aplica en [[Fase 5 - Eventos y Outbox]] y [[Fase 8 - Batch y Cierre de Dia]].

## 🔗 Relaciones
- **Módulo**: [[System Design]]
- **Anterior**: [[05 - Base de Datos a Escala]]
- **Siguiente**: [[07 - Rate Limiting]]
- **Se conecta con**: [[02 - ExecutorService y Thread Pools]], [[07 - Mensajeria Gestionada]], [[08 - Alta Disponibilidad]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

