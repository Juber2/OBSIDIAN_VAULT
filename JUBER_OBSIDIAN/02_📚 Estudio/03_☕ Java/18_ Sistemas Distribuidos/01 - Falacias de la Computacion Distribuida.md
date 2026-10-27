---
tags:
  - Java
  - Distribuido
  - Arquitectura
Creado: 2026-09-08
Relacionado:
  - Sistemas Distribuidos
  - Microservicios
Palabras clave:
  - falacias, latencia, red, timeout, particion, resiliencia, Deutsch
---

## 🧠 Concepto

En los 90, Peter Deutsch y James Gosling (Sun Microsystems) enumeraron las **ocho falacias de la computación distribuida**: suposiciones que todo programador hace sin darse cuenta al pasar de una máquina a varias, y que son todas falsas.

Treinta años después siguen explicando la mayoría de los incidentes en producción.

## 📋 Las ocho

| # | Falacia | Realidad |
|---|---|---|
| 1 | La red es confiable | los paquetes se pierden, las conexiones se cortan |
| 2 | La latencia es cero | una llamada remota es 10.000× más lenta que una local |
| 3 | El ancho de banda es infinito | serializar y transportar cuesta |
| 4 | La red es segura | todo tramo es interceptable |
| 5 | La topología no cambia | los nodos se reemplazan, las IP cambian |
| 6 | Hay un solo administrador | los sistemas los operan equipos distintos |
| 7 | El costo de transporte es cero | serialización, TLS, egreso de datos |
| 8 | La red es homogénea | protocolos, versiones y latencias distintas |

## 1️⃣ La red es confiable

La falacia madre. De ella se derivan casi todas las decisiones de diseño de un sistema distribuido.

```java
// ❌ Esto asume que la llamada o funciona o falla
var respuesta = clienteRemoto.procesar(peticion);
```

El problema no es que falle: es que **puede fallar de una tercera manera** que no existe en una llamada local.

```
Llamada local:   funciona  |  lanza excepción
Llamada remota:  funciona  |  falla  |  ❓ NO SÉ QUÉ PASÓ
```

Un timeout no dice "falló". Dice "no recibí respuesta". La operación pudo:

- no haber llegado
- haber llegado y estar ejecutándose
- haber terminado bien, y perderse la respuesta
- haber fallado del otro lado

> [!danger] Ese cuarto estado es el que rompe todo
> Es el problema del estado desconocido de [[06 - Transferencias y Consistencia]]. Y es por eso que existe la idempotencia: como no podés saber si la operación pasó, tenés que hacer que **repetirla sea inofensivo**.

Es un resultado teórico conocido: el **problema de los dos generales** demuestra que sobre un canal no confiable no hay protocolo que garantice acuerdo con certeza. No es una limitación de implementación — es imposible.

## 2️⃣ La latencia es cero

Los órdenes de magnitud que hay que tener internalizados:

| Operación | Tiempo | En escala humana (×10⁹) |
|---|---|---|
| Referencia a caché L1 | 1 ns | 1 segundo |
| Acceso a RAM | 100 ns | 2 minutos |
| SSD lectura aleatoria | 150 µs | 2 días |
| Red en el mismo datacenter | 500 µs | 6 días |
| Lectura de disco mecánico | 10 ms | 4 meses |
| Red intercontinental | 150 ms | 5 años |

> [!warning] Una llamada remota no es "un método un poco más lento"
> Es cuatro órdenes de magnitud más lenta. Un bucle que llama 100 veces a un servicio remoto no tarda "un poco más": tarda 100 × 500 µs mínimo, más serialización, más TLS, más colas.

```java
// ❌ N llamadas remotas: N × latencia
for (UUID id : ids) {
    resultados.add(clienteRemoto.obtener(id));       // 100 ids = 100 viajes
}

// ✅ una llamada por lote
resultados = clienteRemoto.obtenerVarios(ids);       // 1 viaje
```

Es el **N+1 de red**: el mismo problema que el N+1 de JPA, con latencias mucho peores.

> [!tip] La latencia tiene un piso físico
> Lima a Frankfurt son ~10.000 km. A la velocidad de la luz en fibra (~200.000 km/s), la ida y vuelta son ~100 ms **como mínimo teórico**. Ninguna optimización de código baja de ahí. Si tu SLA exige menos, necesitás presencia local, no mejor código.

## 3️⃣ El ancho de banda es infinito

```java
// ❌ traer todo y filtrar en el cliente
List<Movimiento> todos = clienteRemoto.obtenerTodos();
var recientes = todos.stream().filter(m -> m.fecha().isAfter(ayer)).toList();

// ✅ que filtre y pagine el que tiene los datos
var recientes = clienteRemoto.obtener(desde, hasta, pageable);
```

Y el formato importa: JSON verboso, con nombres largos y campos que nadie usa, multiplica el costo. En rutas de alto volumen se justifican formatos binarios (Protobuf, Avro).

## 4️⃣ La red es segura

> [!danger] Dentro de la red privada no es lo mismo que seguro
> El modelo de "perímetro confiable" asume que si alguien está adentro, es legítimo. Un solo servicio comprometido invalida esa suposición para todo el resto.

**Confianza cero** (zero trust): cada servicio autentica a quien lo llama, aunque el llamante esté en la misma red.

```yaml
# mTLS entre servicios: los dos lados presentan certificado
spring:
  ssl:
    bundle:
      pem:
        interno:
          keystore:   { certificate: /certs/svc.crt, private-key: /certs/svc.key }
          truststore: { certificate: /certs/ca.crt }
```

## 5️⃣ La topología no cambia

En un entorno con contenedores, los nodos son ganado, no mascotas: se crean, se destruyen y cambian de IP constantemente.

```java
// ❌ dirección fija
var cliente = WebClient.create("http://10.0.1.42:8080");

// ✅ resolución dinámica por nombre lógico
@LoadBalanced
var cliente = WebClient.create("http://cuentas-service");
```

> [!warning] Cuidado con la caché de DNS de la JVM
> Históricamente la JVM cacheaba resoluciones DNS **para siempre** cuando había un `SecurityManager`. Si una IP cambia, tu app sigue llamando a la vieja. Configurá `networkaddress.cache.ttl` a un valor corto (30-60 s) en entornos dinámicos.

## 6️⃣ Hay un solo administrador

El servicio del que dependés lo opera otro equipo, con su propio calendario de despliegues, sus propias ventanas de mantenimiento y su propia definición de "urgente".

Consecuencias de diseño:

- Tu servicio debe **degradar**, no caerse, cuando el otro no responde.
- Los contratos se versionan y se mantienen compatibles hacia atrás.
- Necesitás observabilidad propia: no podés depender de que el otro equipo te avise.

## 7️⃣ El costo de transporte es cero

| Costo | Dónde aparece |
|---|---|
| Serialización | CPU en ambos extremos |
| Handshake TLS | latencia adicional en cada conexión nueva |
| Egreso de datos | **factura de la nube**, sobre todo entre regiones |
| Conexiones | memoria y descriptores de archivo |

> [!tip] El tráfico entre zonas de disponibilidad se factura
> Es un costo real y frecuentemente invisible hasta que llega la factura. Distribuir sin criterio puede salir más caro que el cómputo que ahorra.

## 8️⃣ La red es homogénea

Versiones de protocolo distintas, proxies que se comportan distinto, límites de tamaño distintos, timeouts distintos en cada tramo.

> [!danger] Los timeouts en cascada tienen que decrecer hacia adentro
> ```
> Cliente        30 s
>   Gateway      25 s
>     Servicio A 20 s
>       Servicio B 5 s
> ```
> Si el timeout de B fuera mayor que el de A, A se rinde mientras B sigue trabajando: consumís recursos por un resultado que nadie va a leer. Cada capa debe rendirse **antes** que la de afuera.

## 🛡️ Qué se hace con esto

Las falacias no se resuelven: se **diseñan alrededor**.

| Falacia | Mecanismo de defensa |
|---|---|
| Red no confiable | timeouts, reintentos con backoff, **idempotencia**, circuit breaker |
| Latencia | operaciones por lote, caché, asincronía, colocación |
| Ancho de banda | paginación, compresión, formatos binarios, campos selectivos |
| Inseguridad | mTLS, confianza cero, cifrado en tránsito |
| Topología cambiante | descubrimiento de servicios, health checks, TTL de DNS corto |
| Múltiples administradores | contratos versionados, degradación, observabilidad propia |
| Costo | medir egreso, colocar servicios que hablan mucho |
| Heterogeneidad | timeouts en cascada, negociación de contenido |

```java
@Bean
public WebClient clienteResiliente(WebClient.Builder builder) {
    return builder
        .clientConnector(new ReactorClientHttpConnector(
            HttpClient.create()
                .option(CONNECT_TIMEOUT_MILLIS, 2_000)
                .responseTimeout(Duration.ofSeconds(5))))
        .filter(reintentarConBackoff())        // solo operaciones idempotentes
        .filter(circuitBreaker())
        .filter(propagarTraceId())
        .build();
}
```

> [!danger] Nunca reintentes automáticamente una operación no idempotente
> Un reintento sobre un `POST /transferencias` sin clave de idempotencia duplica el dinero. La política de reintentos y la idempotencia son **el mismo diseño**, no dos decisiones separadas.

## 🧪 Cómo se prueba

No alcanza con tests de camino feliz: hay que **inyectar los fallos**.

```java
@Test
void el_servicio_degrada_cuando_el_dependiente_no_responde() {
    servidorSimulado.stubFor(get("/cuentas/1").willReturn(aResponse().withFixedDelay(10_000)));

    var resultado = servicio.obtenerConFallback(id);

    assertTrue(resultado.esDegradado());
    assertFalse(resultado.esError());       // degrada, no se cae
}

@Test
void un_timeout_no_dispara_reintento_en_operaciones_no_idempotentes() {
    servidorSimulado.stubFor(post("/transferencias").willReturn(aResponse().withFixedDelay(10_000)));
    assertThrows(TimeoutException.class, () -> cliente.transferir(req));
    servidorSimulado.verify(1, postRequestedFor(urlEqualTo("/transferencias")));  // ← una sola vez
}
```

Con **Toxiproxy** (integrado con Testcontainers) se simulan latencia, cortes y pérdida de paquetes de forma reproducible.

## ⚠️ Errores Comunes

> [!warning] Los que producen incidentes
> - **Tratar una llamada remota como local**: bucles que hacen N viajes.
> - **Reintentar sin idempotencia**: multiplica el efecto en vez de repararlo.
> - **Timeouts crecientes hacia adentro**: recursos consumidos para nada.
> - **Sin timeout**: un hilo bloqueado indefinidamente agota el pool y tumba el servicio.
> - **Confiar en la red interna**: un servicio comprometido accede a todo.
> - **Direcciones IP fijas** en un entorno con contenedores.
> - **Probar solo el camino feliz**: los fallos aparecen en producción.

## 📌 Notas Rápidas
- Si tuvieras que quedarte con una sola falacia, quedate con la primera: **la red no es confiable, y el estado desconocido es su consecuencia**. Casi todo lo demás del módulo existe para lidiar con eso.
- La tabla de latencias en escala humana ayuda a intuir por qué una llamada remota dentro de un bucle es tan grave: es la diferencia entre esperar un segundo y esperar seis días.
- Antes de distribuir, preguntate si el problema se resuelve sin red. Un monolito modular evita las ocho falacias de una — ver [[07 - Modular Monolith]].

## 🔗 Relaciones
- **Módulo**: [[Sistemas Distribuidos]]
- **Siguiente**: [[02 - CAP PACELC y Consistencia]]
- **Se conecta con**: [[05 - Circuit Breaker - Resilience4j]], [[03 - Idempotencia y Deduplicacion]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

