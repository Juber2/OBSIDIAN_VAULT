---
tags:
  - Java
  - System Design
  - Resiliencia
Creado: 2026-09-08
Relacionado:
  - System Design
  - 06 - Colas y Backpressure
Palabras clave:
  - rate limiting, token bucket, ventana deslizante, 429, Retry-After, cuota, Redis
---

## 🧠 Concepto

Limitar la tasa es controlar **cuántas peticiones acepta el sistema por unidad de tiempo**, por cliente. Sirve para tres cosas distintas que conviene no mezclar:

| Objetivo | Ejemplo |
|---|---|
| **Proteger** el sistema | que un cliente no consuma toda la capacidad |
| **Monetizar** | plan gratuito 100/hora, plan pago 10.000/hora |
| **Seguridad** | frenar fuerza bruta contra el login |

> [!warning] Limitar la tasa no es descartar carga
> El límite de tasa es una **regla por cliente**, conocida y predecible: "tenés 1.000 por hora". El descarte de carga ([[06 - Colas y Backpressure]]) es una **reacción a la saturación**, independiente de quién sea el cliente.
> Hacen falta las dos: el límite evita el abuso; el descarte protege ante un pico legítimo.

## ⚙️ Los cinco algoritmos

### 1 · Ventana fija

```java
// Un contador por ventana de tiempo
String clave = "rl:%s:%d".formatted(clienteId, Instant.now().getEpochSecond() / 60);
Long actual = redis.increment(clave);
if (actual == 1) redis.expire(clave, Duration.ofMinutes(1));
return actual <= limite;
```

| Ventaja | Desventaja |
|---|---|
| Muy simple, poca memoria | **permite el doble del límite en el borde** |

> [!danger] El problema del borde
> ```
> Límite: 100 por minuto
> 12:00:59 → 100 peticiones ✅ (ventana de las 12:00)
> 12:01:01 → 100 peticiones ✅ (ventana de las 12:01)
> = 200 peticiones en 2 segundos
> ```
> Para proteger de picos, la ventana fija no alcanza.

### 2 · Registro de ventana deslizante

```java
// Guarda el timestamp de cada petición
long ahora = System.currentTimeMillis();
redis.zRemoveRangeByScore(clave, 0, ahora - ventanaMs);   // limpia lo viejo
long cantidad = redis.zCard(clave);
if (cantidad < limite) {
    redis.zAdd(clave, ahora, UUID.randomUUID().toString());
    return true;
}
return false;
```

Exacto, sin problema de borde. **Costoso**: guarda un registro por petición.

### 3 · Contador de ventana deslizante

```java
// Aproxima ponderando la ventana anterior y la actual
long ventanaActual  = ahora / ventanaMs;
long enActual       = contador(clave, ventanaActual);
long enAnterior     = contador(clave, ventanaActual - 1);

double pesoAnterior = 1.0 - (double)(ahora % ventanaMs) / ventanaMs;
double estimado     = enAnterior * pesoAnterior + enActual;

return estimado < limite;
```

> [!tip] Es el mejor equilibrio para la mayoría de los casos
> Memoria de ventana fija (dos contadores), precisión cercana a la del registro completo. Es lo que usan muchas implementaciones de producción, incluidas varias CDN.

### 4 · Cubeta de fichas (token bucket)

```java
public class CubetaDeFichas {
    private final long capacidad;          // ráfaga máxima
    private final double fichasPorSegundo; // tasa sostenida
    private double fichas;
    private long ultimaRecarga;

    public synchronized boolean intentarConsumir(int cantidad) {
        recargar();
        if (fichas >= cantidad) { fichas -= cantidad; return true; }
        return false;
    }

    private void recargar() {
        long ahora = System.nanoTime();
        double transcurrido = (ahora - ultimaRecarga) / 1e9;
        fichas = Math.min(capacidad, fichas + transcurrido * fichasPorSegundo);
        ultimaRecarga = ahora;
    }
}
```

> [!tip] Es el algoritmo más usado, y por buenas razones
> Permite **ráfagas** hasta el tamaño de la cubeta y limita la **tasa sostenida** con el ritmo de recarga. Eso refleja el uso real: un cliente que estuvo inactivo puede enviar un pico, pero no puede mantenerlo.
> Es lo que implementan Bucket4j, Guava `RateLimiter` y la mayoría de las puertas de enlace.

### 5 · Cubeta con fuga (leaky bucket)

Procesa a ritmo constante, sin permitir ráfagas. Suaviza el tráfico hacia el sistema de atrás. Se usa cuando lo que hay que proteger no tolera picos en absoluto.

### Comparación

| Algoritmo | Memoria | Precisión | Ráfagas | Uso típico |
|---|---|---|---|---|
| Ventana fija | mínima | baja | ⚠️ 2× en el borde | casos simples |
| Registro deslizante | alta | **exacta** | ❌ | límites estrictos |
| Contador deslizante | baja | alta | ❌ | **APIs, propósito general** |
| **Cubeta de fichas** | baja | alta | ✅ controladas | **el más usado** |
| Cubeta con fuga | baja | alta | ❌ suaviza | proteger sistemas frágiles |

## 💻 Implementación

```xml
<dependency>
    <groupId>com.bucket4j</groupId>
    <artifactId>bucket4j-core</artifactId>
</dependency>
```

```java
@Component
public class LimitadorDeTasa {

    private final Cache<String, Bucket> cubetas = Caffeine.newBuilder()
        .expireAfterAccess(Duration.ofHours(1))
        .maximumSize(100_000)
        .build();

    public boolean permitir(String clienteId, Plan plan) {
        return cubetas.get(clienteId, id -> crear(plan)).tryConsume(1);
    }

    private Bucket crear(Plan plan) {
        return Bucket.builder()
            .addLimit(l -> l.capacity(plan.rafaga())
                            .refillGreedy(plan.porMinuto(), Duration.ofMinutes(1)))
            .build();
    }
}
```

### Distribuido

Con varias instancias, el estado tiene que ser compartido.

```lua
-- Script Lua: atómico en Redis
local clave = KEYS[1]
local limite = tonumber(ARGV[1])
local ventana = tonumber(ARGV[2])

local actual = redis.call('INCR', clave)
if actual == 1 then
    redis.call('EXPIRE', clave, ventana)
end
return actual <= limite and 1 or 0
```

> [!danger] Sin atomicidad, el límite se puede exceder
> ```java
> Long actual = redis.get(clave);      // ← otro hilo puede incrementar acá
> if (actual < limite) redis.increment(clave);
> ```
> La ventana entre leer y escribir permite que varias peticiones pasen la validación a la vez. **El script Lua se ejecuta como una unidad** en el servidor de Redis y elimina la carrera.

```java
// Alternativa: la propia puerta de enlace
// Spring Cloud Gateway con Redis, sin código propio
```

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: cuentas
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 100     # por segundo
                redis-rate-limiter.burstCapacity: 200     # ráfaga
                key-resolver: "#{@resolutorPorCliente}"
```

## 📤 La respuesta

```java
@ExceptionHandler(LimiteExcedidoException.class)
public ResponseEntity<ProblemDetail> limiteExcedido(LimiteExcedidoException ex) {
    var pd = ProblemDetail.forStatus(HttpStatus.TOO_MANY_REQUESTS);
    pd.setTitle("Demasiadas peticiones");
    pd.setDetail("Se excedió el límite de %d por %s".formatted(ex.limite(), ex.ventana()));

    return ResponseEntity.status(429)
        .header("RateLimit-Limit", String.valueOf(ex.limite()))
        .header("RateLimit-Remaining", "0")
        .header("RateLimit-Reset", String.valueOf(ex.segundosHastaReinicio()))
        .header("Retry-After", String.valueOf(ex.segundosHastaReinicio()))
        .body(pd);
}
```

> [!tip] Devolvé las cabeceras siempre, no solo al rechazar
> Un cliente bien hecho lee `RateLimit-Remaining` y se autorregula **antes** de llegar al límite. Si solo informás al rechazar, el cliente descubre el límite chocando contra él.

| Cabecera | Qué dice |
|---|---|
| `RateLimit-Limit` | cuota total |
| `RateLimit-Remaining` | cuánto queda |
| `RateLimit-Reset` | segundos hasta el reinicio |
| `Retry-After` | cuándo reintentar (en el `429`) |

> [!danger] `429` sin `Retry-After` provoca tormentas de reintentos
> El cliente no sabe cuánto esperar, así que reintenta de inmediato. Miles de clientes rechazados reintentando en bucle multiplican la carga justo cuando el sistema está saturado.

## 🎯 Por qué clave limitar

```java
@Bean
public KeyResolver resolutorPorCliente() {
    return exchange -> {
        var auth = exchange.getPrincipal();
        return auth.map(p -> "cliente:" + p.getName())
                   .switchIfEmpty(Mono.just("ip:" + ipReal(exchange)));
    };
}
```

| Clave | Uso | Problema |
|---|---|---|
| Por IP | tráfico anónimo | NAT corporativo comparte IP |
| **Por cliente autenticado** | **APIs** ✅ | requiere autenticación previa |
| Por clave de API | terceros | ídem |
| Por endpoint | proteger operaciones caras | más configuración |
| Global | protección de último recurso | no distingue abuso de uso |

> [!warning] Limitar solo por IP castiga a usuarios legítimos
> Una empresa entera puede salir por una sola IP pública. Un límite por IP los trata como un solo cliente y bloquea a todos por el uso de uno.
> Para tráfico autenticado, la clave correcta es el cliente. La IP queda para lo anónimo.

### Límites por operación

```java
@RateLimit(limite = 5,    ventana = "1m")   // login: protege de fuerza bruta
@RateLimit(limite = 100,  ventana = "1m")   // consultas
@RateLimit(limite = 10,   ventana = "1m")   // transferencias
@RateLimit(limite = 2,    ventana = "1h")   // exportar reporte pesado
```

> [!tip] El costo de la operación define el límite
> Una consulta de saldo cuesta milisegundos; una exportación de dos años de movimientos cuesta minutos de CPU. Aplicar el mismo límite a las dos deja pasar el abuso caro y frena el uso barato.

## 🚦 Dónde aplicarlo

```
Cliente → CDN/WAF        ← límite global, protección DDoS
        → API Gateway    ← límite por cliente ✅ el lugar principal
        → Servicio       ← límite por operación cara
        → Base de datos  ← el pool ya limita implícitamente
```

> [!tip] El gateway es el lugar correcto
> Rechazar ahí cuesta casi nada. Rechazar después de autenticar, consultar la base y tomar un bloqueo desperdició todo ese trabajo.

## 📊 Monitorear

| Métrica | Para qué |
|---|---|
| Peticiones rechazadas por cliente | detectar abuso |
| Clientes cerca del límite | avisar antes de bloquear |
| Rechazos totales | ¿el límite es razonable? |
| Distribución de uso | dimensionar los planes |

> [!warning] Muchos rechazos legítimos = el límite está mal
> Si el 20% de los clientes choca contra el límite todos los días, el límite no está protegiendo: está estorbando. Los límites se calibran con datos de uso real, no con un número elegido al azar.

## 🧪 Tests

```java
@Test
void tras_el_limite_se_devuelve_429() {
    IntStream.range(0, 100).forEach(i -> api.get("/cuentas"));
    assertEquals(429, api.get("/cuentas").statusCode());
}

@Test
void la_respuesta_429_incluye_retry_after() {
    agotarLimite();
    var r = api.get("/cuentas");
    assertNotNull(r.header("Retry-After"));
}

@Test
void el_limite_se_respeta_entre_varias_instancias() {
    // dos instancias contra el mismo Redis: el total no excede el límite
    var exitosas = ejecutarEnParalelo(2, 100, () -> api.get("/cuentas"));
    assertTrue(exitosas <= 100);
}

@Test
void la_cubeta_permite_una_rafaga_y_luego_limita() {
    assertEquals(200, contarExitosasInmediatas());     // ráfaga
    assertEquals(429, api.get("/cuentas").statusCode()); // sostenido
}

@Test
void un_cliente_no_consume_la_cuota_de_otro() { ... }

@Test
void el_login_tiene_un_limite_mas_estricto() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que rompen el propósito
> - **Ventana fija** creyendo que protege de picos: permite el doble en el borde.
> - **Sin atomicidad** en el limitador distribuido: se excede el límite.
> - **Estado en memoria** con varias instancias: el límite real es N veces mayor.
> - **`429` sin `Retry-After`**: tormenta de reintentos.
> - **Cabeceras solo al rechazar**: el cliente no puede autorregularse.
> - **Solo por IP** en tráfico autenticado: castiga a usuarios tras un NAT.
> - **Mismo límite para todas las operaciones.**
> - **Limitar después del trabajo caro** en vez de en el borde.
> - **Límites sin calibrar** con datos reales.
> - **Sin límite estricto en el login**: fuerza bruta libre.

## 📌 Notas Rápidas
- Si tenés que elegir un algoritmo: **cubeta de fichas**. Permite ráfagas razonables, limita el sostenido, usa poca memoria y hay implementaciones maduras en Java (Bucket4j).
- El problema del borde de la ventana fija es una pregunta clásica de entrevista. Poder explicarlo con el ejemplo de 12:00:59 y 12:01:01 demuestra que entendiste el algoritmo, no solo su nombre.
- Devolver las cabeceras `RateLimit-*` en todas las respuestas es un detalle chico que mejora mucho la experiencia de quien integra con tu API.
- Se aplica en [[Fase 6 - Microservicios y SAGA]] y se relaciona con los límites de negocio de [[13 - Antifraude y Limites]].

## 🔗 Relaciones
- **Módulo**: [[System Design]]
- **Anterior**: [[06 - Colas y Backpressure]]
- **Siguiente**: [[08 - Alta Disponibilidad]]
- **Se conecta con**: [[04 - API Gateway]], [[13 - Antifraude y Limites]], [[10 - Red y Exposicion]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

