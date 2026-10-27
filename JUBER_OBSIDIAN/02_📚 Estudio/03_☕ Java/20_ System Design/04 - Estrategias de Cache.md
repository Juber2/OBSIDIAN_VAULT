---
tags:
  - Java
  - System Design
  - Performance
Creado: 2026-09-08
Relacionado:
  - System Design
  - 05 - Cache con Redis y Caffeine
Palabras clave:
  - cache-aside, write-through, invalidacion, estampida, hot key, TTL, LRU, Redis
---

## 🧠 Concepto

Cachear es guardar el resultado de algo caro para no volver a calcularlo. Es la optimización con mejor relación esfuerzo/beneficio que existe — **y la que más bugs sutiles introduce**.

> [!warning] Las dos cosas difíciles
> Hay una frase clásica en computación: los dos problemas difíciles son la invalidación de caché y nombrar cosas. Es un chiste con fondo real: **agregar un caché es trivial, mantenerlo correcto no**.

## 🔀 Los patrones

### Cache-aside (lazy loading)

El más usado. La aplicación gestiona el caché explícitamente.

```java
public Cuenta obtener(UUID id) {
    Cuenta cacheada = cache.get(id);
    if (cacheada != null) return cacheada;         // acierto

    Cuenta cuenta = repo.findById(id).orElseThrow(); // fallo → base
    cache.put(id, cuenta, Duration.ofMinutes(5));
    return cuenta;
}
```

```java
// Con Spring, lo mismo declarativo
@Cacheable(value = "cuentas", key = "#id")
public Cuenta obtener(UUID id) { return repo.findById(id).orElseThrow(); }

@CacheEvict(value = "cuentas", key = "#cuenta.id")
public void actualizar(Cuenta cuenta) { repo.save(cuenta); }
```

| Ventaja | Desventaja |
|---|---|
| Solo se cachea lo que se usa | el primer acceso siempre es lento |
| Si el caché muere, el sistema sigue | puede servir datos viejos |
| Simple de razonar | la invalidación es manual |

### Write-through y write-behind

```java
// Write-through: escribe en los dos, sincrónicamente
public void actualizar(Cuenta cuenta) {
    repo.save(cuenta);
    cache.put(cuenta.id(), cuenta);       // el caché nunca queda viejo
}

// Write-behind: escribe en el caché y difiere la base
public void registrarVisita(UUID id) {
    cache.increment("visitas:" + id);      // la base se actualiza en lote después
}
```

> [!danger] Write-behind puede perder datos
> Si el caché muere antes de volcar a la base, lo que estaba pendiente se perdió. Sirve para contadores de visitas o métricas aproximadas; **jamás para dinero**.

### Refresh-ahead

```java
Caffeine.newBuilder()
    .expireAfterWrite(Duration.ofMinutes(10))
    .refreshAfterWrite(Duration.ofMinutes(5))   // refresca en segundo plano
    .build(this::cargar);
```

Refresca antes de que expire, de forma asíncrona: el usuario nunca espera la recarga. Ideal para datos consultados con frecuencia y que cambian poco — tipos de cambio, catálogos, configuración.

## 🏢 Los niveles

```
Navegador  →  CDN  →  Caché local (Caffeine)  →  Caché distribuido (Redis)  →  Base
   ms          ms              ns                      < 1 ms                 10 ms
```

| Nivel | Latencia | Compartido | Uso |
|---|---|---|---|
| Local en proceso | ~100 ns | ❌ por instancia | datos pequeños y muy leídos |
| Distribuido (Redis) | 0,5-2 ms | ✅ | sesiones, saldos, resultados |
| CDN | 10-50 ms | ✅ global | estáticos, respuestas públicas |

> [!warning] El caché local no es consistente entre instancias
> Con 10 instancias hay 10 copias que pueden divergir. Invalidar en una no invalida las otras. Es aceptable para datos que toleran unos segundos de desfase (configuración, catálogos) e inaceptable para saldos.

```java
// Caché de dos niveles: local muy corto + distribuido
@Bean
public CacheManager cacheManager(RedisConnectionFactory factory) {
    var local = Caffeine.newBuilder().maximumSize(10_000)
        .expireAfterWrite(Duration.ofSeconds(30));      // ← ventana chica de inconsistencia
    // ...combinado con Redis para el nivel compartido
}
```

## ⏳ Expiración y desalojo

| Política | Cómo elige qué borrar |
|---|---|
| **TTL** | por tiempo desde la escritura |
| **LRU** | el menos usado recientemente |
| **LFU** | el menos usado en total |
| FIFO | el más antiguo |
| Aleatorio | uno cualquiera |

```java
Caffeine.newBuilder()
    .maximumSize(10_000)                          // desalojo por cantidad
    .expireAfterWrite(Duration.ofMinutes(5))       // TTL
    .recordStats()                                 // ← métricas: imprescindible
    .build();
```

> [!tip] El TTL es el mecanismo de invalidación más robusto
> Toda la lógica de invalidación explícita puede fallar: alguien agrega una ruta de escritura y olvida invalidar. El TTL garantiza que **el dato viejo tiene fecha de vencimiento** aunque el código tenga bugs.
> Regla práctica: siempre poné TTL, incluso cuando invalidás explícitamente. Es la red de seguridad.

## 💥 La estampida

El problema de concurrencia clásico del caché.

```
Clave popular expira
  → 1.000 peticiones simultáneas fallan el caché
  → 1.000 consultas idénticas a la base al mismo tiempo
  → la base colapsa 💥
```

### Solución 1 — Un solo cargador

```java
// Caffeine lo hace por defecto con LoadingCache:
// si 1.000 hilos piden la misma clave ausente, solo UNO ejecuta la carga
LoadingCache<UUID, Cuenta> cache = Caffeine.newBuilder()
    .expireAfterWrite(Duration.ofMinutes(5))
    .build(id -> repo.findById(id).orElseThrow());
```

```java
// En caché distribuido: candado sobre la clave
public Cuenta obtener(UUID id) {
    Cuenta cacheada = redis.get(clave(id));
    if (cacheada != null) return cacheada;

    String candado = "lock:" + id;
    if (redis.setIfAbsent(candado, "1", Duration.ofSeconds(10))) {
        try {
            Cuenta cuenta = repo.findById(id).orElseThrow();
            redis.set(clave(id), cuenta, TTL);
            return cuenta;
        } finally {
            redis.delete(candado);
        }
    }
    esperarBreve();
    return obtener(id);        // reintenta: probablemente ya esté en caché
}
```

### Solución 2 — Expiración anticipada probabilística

```java
// Cada lector tiene una probabilidad creciente de refrescar ANTES de que expire
public boolean debeRefrescar(Instant expiracion, Duration tiempoDeCarga) {
    long restante = Duration.between(Instant.now(), expiracion).toMillis();
    double umbral = tiempoDeCarga.toMillis() * BETA * Math.log(ThreadLocalRandom.current().nextDouble());
    return restante < -umbral;
}
```

Los refrescos se dispersan en el tiempo en vez de concentrarse en el instante de expiración.

### Solución 3 — Servir viejo mientras se revalida

```java
// Devolver el dato vencido y refrescar en segundo plano
if (entrada.estaVencida() && !entrada.estaMuyVencida()) {
    ejecutor.submit(() -> refrescar(clave));
    return entrada.valor();        // el usuario no espera
}
```

> [!tip] Dispersá los TTL
> Si cargás 10.000 entradas al arrancar todas con el mismo TTL, expiran todas juntas. Agregá una variación aleatoria:
> ```java
> Duration ttl = BASE.plusSeconds(ThreadLocalRandom.current().nextInt(60));
> ```

## 🔥 Claves calientes

```
Una cuenta corporativa recibe el 40% de las consultas
→ esa clave vive en un solo nodo de Redis
→ ese nodo se satura mientras los demás están ociosos
```

| Mitigación | Cómo |
|---|---|
| Caché local delante | las claves calientes se sirven en proceso |
| Replicar la clave | `clave:0`, `clave:1`... y elegir al azar |
| Partición dedicada | aislar el tráfico caliente |

> [!tip] La caché local resuelve las claves calientes casi sola
> Una entrada muy consultada se sirve desde memoria del proceso, sin tocar la red ni Redis. Es la razón principal para tener un nivel local además del distribuido.

## 🗑️ Invalidación

```java
// 1 · Por TTL: simple y robusto
cache.put(clave, valor, Duration.ofMinutes(5));

// 2 · Explícita al escribir
@CacheEvict(value = "cuentas", key = "#cuenta.id")
public void actualizar(Cuenta cuenta) { repo.save(cuenta); }

// 3 · Por evento: invalida en TODAS las instancias
@KafkaListener(topics = "cuentas-modificadas")
public void alModificarse(CuentaModificada evento) {
    cacheLocal.invalidate(evento.cuentaId());
}

// 4 · Por versión: no invalida, cambia la clave
String clave = "cuenta:%s:v%d".formatted(id, version);
```

> [!danger] Invalidar y escribir en la base no es atómico
> ```
> 1 · borra del caché    ✅
> 2 · escribe en base    💥 falla
> → el caché se recarga con el valor viejo desde la base
> ```
> Se mitiga escribiendo primero y borrando el caché después, con TTL corto como red de seguridad. **La consistencia perfecta entre caché y base no existe**: se elige una ventana de inconsistencia tolerable.

> [!tip] La invalidación por versión evita el problema
> Si la clave incluye la versión del dato, no hay nada que invalidar: la escritura cambia la versión y la clave vieja simplemente deja de consultarse y expira sola.

## 🚫 Caché negativo

```java
// Sin esto, mil peticiones a un id inexistente son mil consultas a la base
public Optional<Cuenta> buscar(UUID id) {
    if (cache.contieneAusencia(id)) return Optional.empty();

    var cuenta = repo.findById(id);
    if (cuenta.isEmpty()) {
        cache.marcarAusente(id, Duration.ofSeconds(30));   // TTL corto
    }
    return cuenta;
}
```

> [!warning] TTL corto para las ausencias
> Un id que hoy no existe puede existir en un minuto. Cachear la ausencia mucho tiempo hace que un dato recién creado parezca no existir.

## 📊 Medir

```java
Caffeine.newBuilder().recordStats().build();

CacheStats stats = cache.stats();
stats.hitRate();          // ← la métrica principal
stats.evictionCount();
stats.averageLoadPenalty();
```

| Métrica | Objetivo |
|---|---|
| Tasa de aciertos | > 80% para que valga la pena |
| Desalojos | si es alta, falta tamaño |
| Latencia de carga | cuánto ahorra cada acierto |
| Uso de memoria | que no cause presión de GC |

> [!danger] Un caché con 30% de aciertos empeora el sistema
> Pagás el costo de consultarlo y de mantenerlo, y el 70% de las veces igual vas a la base. Si la tasa de aciertos es baja, la respuesta no es agrandar el caché: es entender **por qué** no hay reúso. Quizá los datos no se repiten y no había nada que cachear.

## 🎯 Qué cachear y qué no

| Cachear ✅ | No cachear ❌ |
|---|---|
| Consultas caras y repetidas | datos que cambian en cada lectura |
| Datos que cambian poco | información crítica en tiempo real |
| Configuración y catálogos | resultados de una sola consulta |
| Resultados de agregaciones | datos con requisitos legales de exactitud |
| Sesiones | operaciones de escritura |

> [!danger] El saldo antes de una transferencia no se lee del caché
> Un saldo cacheado puede estar desactualizado y habilitar un doble gasto. La validación va contra la base primaria, con bloqueo — ver [[06 - Transferencias y Consistencia]].
> Sí podés cachear el saldo **para mostrar** en pantalla, con TTL corto. Son dos usos distintos del mismo dato, con requisitos distintos.

## 🧪 Tests

```java
@Test
void la_segunda_consulta_no_toca_la_base() {
    servicio.obtener(id);
    servicio.obtener(id);
    verify(repo, times(1)).findById(id);
}

@Test
void actualizar_invalida_la_entrada() {
    servicio.obtener(id);
    servicio.actualizar(cuentaModificada);
    servicio.obtener(id);
    verify(repo, times(2)).findById(id);
}

@Test
void mil_peticiones_concurrentes_generan_una_sola_carga() {
    ejecutarEnParalelo(1000, () -> servicio.obtener(id));
    verify(repo, times(1)).findById(id);        // sin estampida
}

@Test
void si_el_cache_esta_caido_el_sistema_sigue_funcionando() {
    redis.detener();
    assertDoesNotThrow(() -> servicio.obtener(id));
}

@Test
void la_validacion_de_saldo_no_usa_cache() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que introducen bugs sutiles
> - **Cachear sin TTL**: sin red de seguridad ante fallos de invalidación.
> - **Sin protección contra estampida**: la expiración de una clave popular tumba la base.
> - **TTL uniforme**: expiran todas juntas.
> - **Caché local para datos que deben ser consistentes** entre instancias.
> - **Sin métricas de tasa de aciertos**: no sabés si sirve.
> - **Cachear el saldo para validar operaciones.**
> - **Que el sistema falle si el caché no responde**: debe ser una optimización, no una dependencia.
> - **Sin caché negativo**: los ids inexistentes golpean la base.
> - **Write-behind para datos que no se pueden perder.**

## 📌 Notas Rápidas
- La pregunta antes de agregar un caché: **¿qué pasa si el dato está 5 minutos viejo?** Si la respuesta es "nada", cacheá. Si es "un cliente gasta dos veces", no.
- Un caché tiene que ser una **optimización, no una dependencia**: si Redis se cae, el sistema debe seguir funcionando más lento, no fallar.
- La estampida es el problema de caché que más aparece en entrevistas de diseño. Saber nombrarla y proponer las tres soluciones es una respuesta completa.
- Se aplica en [[Fase 8 - Batch y Cierre de Dia]] y se detalla en [[05 - Cache con Redis y Caffeine]].

## 🔗 Relaciones
- **Módulo**: [[System Design]]
- **Anterior**: [[03 - APIs a Escala]]
- **Siguiente**: [[05 - Base de Datos a Escala]]
- **Se conecta con**: [[05 - Cache con Redis y Caffeine]], [[02 - CAP PACELC y Consistencia]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

