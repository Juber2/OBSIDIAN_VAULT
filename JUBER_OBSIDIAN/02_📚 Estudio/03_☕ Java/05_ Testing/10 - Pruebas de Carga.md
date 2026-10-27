---
tags:
  - Java
  - Testing
  - Performance
Creado: 2026-09-08
Relacionado:
  - Testing en Java
  - Performance Java
Palabras clave:
  - k6, Gatling, JMeter, p99, omision coordinada, modelo abierto, soak, spike
---

## 🧠 Concepto

Una prueba de carga responde una pregunta que ningún otro test responde: **¿el sistema aguanta el volumen esperado, y qué se rompe primero cuando no aguanta?**

No sirve para encontrar bugs de lógica: sirve para encontrar el **cuello de botella**, que casi nunca está donde uno cree.

## 🎭 Los cinco tipos

| Tipo | Qué busca | Duración |
|---|---|---|
| **Humo** (smoke) | que el escenario funcione | 1-2 min, 1-5 usuarios |
| **Carga** (load) | comportamiento con el volumen esperado | 10-30 min |
| **Estrés** (stress) | dónde se rompe | hasta que falle |
| **Remojo** (soak) | fugas y degradación lenta | 2-8 horas |
| **Pico** (spike) | reacción a un salto brusco | minutos |

> [!tip] El de remojo es el que más gente saltea y el que más encuentra
> Fugas de memoria, conexiones que no se devuelven al pool, cachés que crecen sin límite, archivos que no se cierran. Nada de eso aparece en 10 minutos: aparece a las tres horas, que es cuando el sistema lleva medio día en producción.

## 🚀 k6

```javascript
// carga.js
import http from 'k6/http';
import { check } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const errores = new Rate('errores_de_negocio');
const latenciaTransferencia = new Trend('latencia_transferencia');

export const options = {
  scenarios: {
    transferencias: {
      executor: 'ramping-arrival-rate',      // ← modelo ABIERTO
      startRate: 10,
      timeUnit: '1s',
      preAllocatedVUs: 50,
      maxVUs: 500,
      stages: [
        { duration: '2m', target: 50 },      // rampa
        { duration: '5m', target: 50 },      // sostenido
        { duration: '2m', target: 200 },     // pico
        { duration: '3m', target: 200 },
        { duration: '2m', target: 0 },       // bajada
      ],
    },
  },
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1500'],
    http_req_failed: ['rate<0.01'],
    errores_de_negocio: ['rate<0.05'],
  },
};

export default function () {
  const payload = JSON.stringify({
    cuentaOrigen: cuentaAleatoria(),
    cuentaDestino: cuentaAleatoria(),
    monto: 100.00,
    moneda: 'PEN',
  });

  const res = http.post(`${__ENV.URL}/api/v1/transferencias`, payload, {
    headers: {
      'Content-Type': 'application/json',
      'Idempotency-Key': uuidv4(),           // ← clave nueva por operación
      'Authorization': `Bearer ${__ENV.TOKEN}`,
    },
    tags: { operacion: 'transferencia' },
  });

  latenciaTransferencia.add(res.timings.duration);
  errores.add(res.status >= 400);

  check(res, {
    'creada': (r) => r.status === 201,
    'devuelve saldo': (r) => r.json('saldoResultante') !== undefined,
  });
}
```

```bash
k6 run -e URL=https://staging.corebank.com -e TOKEN=$TOKEN carga.js
```

## ⚖️ Modelo abierto vs cerrado

La decisión más importante y la que más se ignora.

| | Cerrado (usuarios virtuales) | Abierto (tasa de llegada) |
|---|---|---|
| Cómo funciona | N usuarios en bucle: piden, esperan respuesta, repiten | llegan X peticiones por segundo, sin importar si las anteriores terminaron |
| Si el sistema se lentifica | **la carga baja sola** | la carga se mantiene |
| Refleja | usuarios con reintento humano | tráfico real de internet |
| Ejecutor en k6 | `ramping-vus` | `ramping-arrival-rate` ✅ |

> [!danger] El modelo cerrado esconde el problema
> Con 100 usuarios virtuales en bucle, si el sistema se degrada de 100 ms a 2 segundos, la tasa de peticiones **cae automáticamente** de 1.000/s a 50/s. El sistema deja de recibir carga justo cuando empieza a fallar, y el test muestra latencias tolerables.
> En producción no pasa eso: los usuarios siguen llegando al mismo ritmo aunque el sistema esté lento, y la cola crece hasta el colapso.
>
> **Para simular tráfico real, usá modelo abierto.**

## ⏱️ Omisión coordinada

El sesgo de medición que invalida la mayoría de las pruebas de carga.

```
El generador quiere enviar 100 peticiones/s (una cada 10 ms).
La petición #50 tarda 1 segundo.
→ Durante ese segundo NO envía las 100 peticiones que correspondían.
→ Esas 100 peticiones lentas nunca se midieron.
→ El p99 reportado ignora el peor momento del test.
```

> [!danger] Es el problema que hace que un p99 reportado sea optimista por órdenes de magnitud
> Lo describió Gil Tene y afecta a casi todas las herramientas configuradas de forma ingenua. La latencia que reportan es "cuánto tardó la respuesta desde que la mandé", no "cuánto habría esperado un usuario que llegó en ese momento".

**Cómo se evita:**

```javascript
// ✅ El modelo abierto (arrival-rate) mantiene la tasa aunque el sistema se demore
executor: 'constant-arrival-rate',
rate: 100,
timeUnit: '1s',
preAllocatedVUs: 100,
maxVUs: 1000,        // ← si el sistema se lentifica, k6 asigna más VUs para sostener la tasa
```

> [!warning] `maxVUs` tiene que ser generoso
> Si el sistema se degrada y k6 no tiene VUs disponibles para mantener la tasa, vuelve a caer en omisión coordinada. k6 avisa con `dropped_iterations`: si ese valor es alto, el test está midiendo mal.

## 🏹 Gatling

Alternativa nativa de la JVM, con DSL en Java o Scala.

```java
public class TransferenciaSimulation extends Simulation {

    HttpProtocolBuilder http = http
        .baseUrl(System.getenv("URL"))
        .acceptHeader("application/json");

    ScenarioBuilder escenario = scenario("Transferencias")
        .exec(http("crear transferencia")
            .post("/api/v1/transferencias")
            .header("Idempotency-Key", session -> UUID.randomUUID().toString())
            .body(StringBody(session -> cuerpoAleatorio()))
            .check(status().is(201))
            .check(jsonPath("$.saldoResultante").exists()));

    {
        setUp(escenario.injectOpen(               // ← modelo abierto
                rampUsersPerSec(10).to(200).during(Duration.ofMinutes(5)),
                constantUsersPerSec(200).during(Duration.ofMinutes(10))))
            .protocols(http)
            .assertions(
                global().responseTime().percentile(95).lt(500),
                global().responseTime().percentile(99).lt(1500),
                global().failedRequests().percent().lt(1.0));
    }
}
```

| | k6 | Gatling | JMeter |
|---|---|---|---|
| Lenguaje | JavaScript | Java / Scala | XML o GUI |
| Consumo del generador | bajo (Go) | medio (JVM) | alto |
| Modelo abierto | ✅ nativo | ✅ | limitado |
| Curva de aprendizaje | baja | media | media |
| Integración en CI | excelente | buena | aceptable |

> [!tip] Si el equipo es de Java, Gatling encaja mejor
> Los escenarios se escriben en Java, van al mismo repositorio y se revisan como cualquier otro código. k6 gana en consumo del generador y simplicidad; para cargas muy altas desde una sola máquina, importa.

## 📊 Qué medir mientras corre

La prueba genera carga; el diagnóstico viene de mirar **el sistema bajo prueba**.

| Señal | Dónde | Síntoma de |
|---|---|---|
| p95 / p99 | la herramienta | degradación real |
| Tasa de error | la herramienta | saturación |
| `hikaricp.connections.pending` | Micrometer | **pool subdimensionado** |
| `jvm.gc.pause` | Micrometer | heap mal configurado |
| CPU del contenedor | plataforma | límite de cómputo |
| Consultas lentas | `pg_stat_statements` | índice faltante |
| Bloqueos | `pg_locks` | contención |
| Profundidad de colas | métricas propias | consumidores insuficientes |

> [!warning] El cuello de botella casi nunca está donde uno cree
> El orden habitual de culpables en una aplicación Spring típica es: **consulta sin índice → pool de conexiones → N+1 → serialización → GC**. El código Java rara vez encabeza la lista.
> Sin observabilidad durante la prueba, solo sabés que se degradó, no por qué — ver [[09 - Observabilidad Cloud]].

## 🎯 Diseñar el escenario

```javascript
// ❌ Todos los usuarios virtuales golpean la MISMA cuenta
const CUENTA = 'ACC-001';

// ✅ Distribución realista
import { SharedArray } from 'k6/data';
const cuentas = new SharedArray('cuentas', () => JSON.parse(open('./cuentas.json')));

function cuentaAleatoria() {
  return cuentas[Math.floor(Math.random() * cuentas.length)];
}
```

> [!danger] Un escenario poco realista mide otra cosa
> Con todos los usuarios sobre la misma cuenta, medís la contención de un bloqueo pesimista, no la capacidad del sistema. Vas a "descubrir" que aguanta 20 QPS cuando en realidad aguanta 500 con tráfico distribuido.
> Y al revés: sin ninguna concentración, no descubrís el problema de la cuenta caliente, que es real.

**Un escenario realista incluye:**
- Distribución de operaciones parecida a la real (16:1 lectura/escritura, por ejemplo).
- Mezcla de cuentas con algo de concentración (regla 80/20).
- Datos de prueba con volumen realista: los índices se comportan distinto con 1.000 filas que con 10 millones.
- Los picos previsibles del negocio (fin de mes, día de pago).

## 🔬 Datos de prueba

> [!danger] Probar contra una base con 1.000 filas no prueba nada
> Todas las consultas son rápidas cuando la tabla entera entra en memoria. Los problemas de índices, paginación y N+1 aparecen con millones de filas.
> El conjunto de datos de prueba tiene que ser del orden de magnitud del real.

```sql
-- Generar volumen realista
INSERT INTO apunte (asiento_id, cuenta_id, monto, moneda, fecha)
SELECT gen_random_uuid(),
       (SELECT id FROM cuenta ORDER BY random() LIMIT 1),
       (random() * 1000)::numeric(19,4),
       'PEN',
       now() - (random() * interval '365 days')
FROM generate_series(1, 10000000);

ANALYZE apunte;      -- ⚠️ imprescindible: sin estadísticas, el plan es malo
```

## ⚙️ En el pipeline

```yaml
- name: Prueba de humo (en cada PR)
  run: k6 run --vus 5 --duration 1m carga.js

- name: Prueba de carga (nocturna)
  run: k6 run --out json=resultados.json carga.js

- name: Comparar contra la línea base
  run: node comparar.js resultados.json baseline.json --tolerancia 10
```

> [!tip] Comparar contra una línea base detecta regresiones graduales
> Un p95 de 480 ms "pasa" el umbral de 500 ms. Pero si la semana pasada era 200 ms, algo se degradó y el umbral absoluto no lo ve.
> Guardar la línea base y alertar ante desvíos porcentuales detecta el problema mientras todavía es chico.

## 🧪 Verificar además de medir

```javascript
check(res, {
  'estado 201': (r) => r.status === 201,
  'el saldo bajó exactamente el monto': (r) => {
    const saldo = r.json('saldoResultante');
    return saldo !== undefined && saldo === saldoEsperado;
  },
});
```

```java
@Test
void tras_la_prueba_de_carga_el_sistema_cuadra() {
    // La invariante contable debe cumplirse tras miles de operaciones concurrentes
    assertEquals(0, libro.totalGlobalPorMoneda().stream()
        .mapToInt(d -> d.suma().signum()).sum());
}
```

> [!tip] Verificar la corrección después de la carga es tan valioso como medir la latencia
> Una prueba de carga es la mejor forma de encontrar condiciones de carrera: miles de operaciones concurrentes reales. Si al terminar el sistema no cuadra, encontraste un bug de concurrencia que ningún test unitario iba a mostrar.

## ⚠️ Errores Comunes

> [!warning] Los que invalidan la prueba
> - **Modelo cerrado** para simular tráfico de internet: esconde la degradación.
> - **Omisión coordinada**: el p99 reportado es optimista por órdenes de magnitud.
> - **`maxVUs` insuficiente**: vuelve a caer en omisión coordinada.
> - **Datos de prueba de juguete**: todo anda con 1.000 filas.
> - **Sin `ANALYZE`** tras generar datos: los planes de consulta son malos.
> - **Todos los usuarios sobre el mismo registro**: medís contención, no capacidad.
> - **Sin observabilidad durante la prueba**: sabés que se degradó, no por qué.
> - **Solo umbrales absolutos**: no detecta regresiones graduales.
> - **Probar en un ambiente muy distinto** al de producción.
> - **No verificar la corrección** después de la carga.

## 📌 Notas Rápidas
- Las dos ideas que más gente desconoce son **modelo abierto vs cerrado** y **omisión coordinada**. Mencionarlas en una entrevista demuestra que hiciste pruebas de carga de verdad, no que corriste JMeter una vez.
- La prueba de remojo es la que encuentra las fugas, y es la que casi nadie corre porque tarda horas. Programarla semanalmente por la noche cuesta poco y encuentra mucho.
- Verificar la invariante contable después de una prueba de carga es una forma excelente de detectar condiciones de carrera — ver [[Fase 8 - Batch y Cierre de Dia]].
- Para medir métodos individuales, esto no sirve: ahí va **JMH** — ver [[06 - Benchmarking con JMH]].

## 🔗 Relaciones
- **Módulo**: [[Testing en Java]] — última nota del módulo
- **Anterior**: [[09 - Mutation Testing]]
- **Se conecta con**: [[Performance Java]], [[01 - Profiling y Diagnostico]], [[06 - Colas y Backpressure]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

