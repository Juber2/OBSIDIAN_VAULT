---
tags:
  - Java
  - Patrones
  - No-GoF
  - Creacional
Creado: 2026-09-10
Relacionado:
  - Patrones de Diseño
Palabras clave:
  - object pool, pool de conexiones, HikariCP, reutilizacion, fuga de recursos, thread pool
---

## 🧠 Concepto

Mantiene un conjunto de objetos **caros de crear** listos para reutilizar, en vez de crearlos y destruirlos cada vez.

**Cuándo usarlo:** casi nunca lo vas a implementar — pero lo usás todos los días. Importa entenderlo porque **el pool de conexiones es el cuello de botella más común de una aplicación Spring**.

## Por qué existe

```
Abrir una conexión a PostgreSQL:
  handshake TCP        →  ~1 ms
  autenticación        →  ~5 ms
  configuración sesión →  ~2 ms
  ────────────────────────────
  total                →  ~10 ms

Una consulta simple:  ~1 ms

Sin pool: cada consulta cuesta 11 ms, de los cuales 10 son puro trámite.
```

Con 500 peticiones por segundo, abrir y cerrar conexiones consume más recursos que el trabajo real — y la base tiene un límite duro de conexiones simultáneas.

## La idea

```java
public class PoolSimple<T> {

    private final BlockingQueue<T> disponibles;
    private final Supplier<T> fabrica;

    public PoolSimple(Supplier<T> fabrica, int tamano) {
        this.fabrica = fabrica;
        this.disponibles = new ArrayBlockingQueue<>(tamano);
        for (int i = 0; i < tamano; i++) disponibles.offer(fabrica.get());
    }

    public T tomar(Duration espera) throws InterruptedException {
        T objeto = disponibles.poll(espera.toMillis(), MILLISECONDS);
        if (objeto == null) throw new PoolAgotadoException();     // ← timeout, no espera infinita
        return objeto;
    }

    public void devolver(T objeto) {
        reiniciar(objeto);                                        // ← limpiar el estado
        disponibles.offer(objeto);
    }
}
```

> [!danger] Los dos errores que convierten un pool en un problema
> 1. **Esperar sin timeout**: si el pool se agota, los hilos quedan bloqueados para siempre y el servicio deja de responder.
> 2. **No reiniciar el estado al devolver**: el siguiente que lo tome hereda datos del anterior. En una conexión eso significa una transacción abierta o variables de sesión ajenas.

## HikariCP — el que usás en Spring Boot

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10           # ← el número que más importa
      minimum-idle: 10
      connection-timeout: 3000        # cuánto espera antes de fallar
      idle-timeout: 600000
      max-lifetime: 1800000           # recicla conexiones viejas
      leak-detection-threshold: 60000 # avisa si alguien no devuelve
```

### Dimensionar el pool

```
conexiones_totales = instancias × maximum-pool-size ≤ límite de la base
```

> [!warning] Un pool grande no acelera nada
> Una base con 8 núcleos no procesa 200 consultas en paralelo: las encola. Un pool grande solo mueve la cola del cliente al servidor, donde cada conexión además consume memoria.
> La fórmula de referencia de HikariCP es `núcleos × 2 + husos de disco` — bastante más chica de lo que la intuición sugiere. Para la mayoría de las aplicaciones, **10 está bien**.

```
Ejemplo:
  4 instancias × 10 conexiones = 40
  PostgreSQL con max_connections = 100  ✅ hay margen

  20 instancias × 50 = 1000  ❌ la base rechaza conexiones
```

### La métrica que hay que mirar

```java
hikaricp.connections.active      // en uso
hikaricp.connections.idle        // libres
hikaricp.connections.pending     // ← HILOS ESPERANDO: si es > 0 sostenido, hay problema
hikaricp.connections.timeout     // peticiones que fallaron por agotamiento
```

> [!danger] `pending > 0` significa que el pool es tu cuello de botella
> Los hilos están esperando una conexión en vez de trabajar. Antes de agrandar el pool, verificá **por qué** las conexiones no se liberan: casi siempre es una consulta lenta o una transacción demasiado larga.
> Agrandar el pool sin diagnosticar solo traslada el problema a la base.

## 💥 La fuga de conexiones

Es el bug más caro relacionado con pools.

```java
// ❌ Si falla, la conexión NUNCA vuelve al pool
Connection conn = dataSource.getConnection();
var stmt = conn.prepareStatement(sql);
var rs = stmt.executeQuery();
procesar(rs);
conn.close();                          // no se ejecuta si procesar() lanza excepción
```

```java
// ✅ try-with-resources cierra siempre
try (Connection conn = dataSource.getConnection();
     var stmt = conn.prepareStatement(sql);
     var rs = stmt.executeQuery()) {
    procesar(rs);
}
```

```
Sin cerrar: cada error deja una conexión perdida.
Con 10 conexiones y un error frecuente, en minutos el pool está vacío
y la aplicación deja de responder — aunque la base esté perfecta.
```

> [!tip] `leak-detection-threshold` te avisa
> Si una conexión está tomada más de N milisegundos, HikariCP loguea el stacktrace de **quién la pidió**. Es la forma más rápida de encontrar la fuga.
> En producción se suele poner en 60000 (1 minuto): las transacciones legítimas nunca duran tanto.

> [!danger] Nunca llames a un servicio externo con una conexión tomada
> ```java
> @Transactional
> public void procesar(UUID id) {
>     var entidad = repo.findById(id).orElseThrow();
>     var respuesta = servicioExterno.validar(entidad);   // 💥 2 segundos con la conexión ocupada
>     entidad.aplicar(respuesta);
> }
> ```
> Con un pool de 10 y una llamada de 2 segundos, 10 peticiones simultáneas dejan el pool vacío. La solución es sacar la llamada externa fuera de la transacción.

## Otros pools que ya usás

| Pool | Qué reutiliza | Configuración clave |
|---|---|---|
| **HikariCP** | conexiones a la base | `maximum-pool-size` |
| `ThreadPoolExecutor` | hilos | tamaño + **cola acotada** |
| Tomcat / Undertow | hilos de atención HTTP | `server.tomcat.threads.max` |
| Apache HttpClient | conexiones HTTP | `maxTotal`, `defaultMaxPerRoute` |
| Jedis / Lettuce | conexiones a Redis | `max-active` |

```java
// Un pool de hilos ES un object pool
var ejecutor = new ThreadPoolExecutor(10, 20, 60L, SECONDS,
    new LinkedBlockingQueue<>(1000),                  // ⚠️ acotada, ver [[06 - Colas y Backpressure]]
    new ThreadPoolExecutor.CallerRunsPolicy());
```

## 🎯 Cuándo NO implementarlo

> [!warning] Con objetos baratos, el pool es más lento que crear
> Crear un objeto en Java moderno cuesta nanosegundos, y el recolector generacional está optimizado para objetos de vida corta. Un pool agrega sincronización, gestión de estado y riesgo de fuga.
> **Solo se justifica cuando crear el objeto implica E/S**: una conexión, un hilo del sistema operativo, un socket.

| Objeto | ¿Pool? |
|---|---|
| Conexión a base o a Redis | ✅ |
| Hilo | ✅ |
| Conexión HTTP | ✅ |
| Un `String` o un DTO | ❌ |
| Un objeto de dominio | ❌ |
| `SimpleDateFormat` | ⚠️ mejor `DateTimeFormatter`, que es inmutable |

## 📌 Notas Rápidas
- **No vas a escribir un pool; vas a configurar uno.** Y el número que más impacta en el rendimiento de una aplicación Spring es `maximum-pool-size`.
- El error de intuición más frecuente: creer que un pool más grande es más rápido. Casi siempre es al revés — ver [[05 - Base de Datos a Escala]].
- `hikaricp.connections.pending > 0` es la métrica que delata el problema. Si aparece, buscá consultas lentas o transacciones largas antes de agrandar nada.
- Las fugas de conexión se previenen con `try-with-resources` y se detectan con `leak-detection-threshold`.
- No es GoF, aunque se lo suele contar entre los creacionales. Es un patrón de gestión de recursos.
- Se relaciona con [[06 - Colas y Backpressure]] (los pools de hilos son pools de objetos) y con [[02 - JDBC y Connection Pools]].

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->
