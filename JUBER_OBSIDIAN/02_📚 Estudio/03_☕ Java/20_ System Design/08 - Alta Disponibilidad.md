---
tags:
  - Java
  - System Design
  - Resiliencia
Creado: 2026-09-08
Relacionado:
  - System Design
  - 05 - Circuit Breaker - Resilience4j
Palabras clave:
  - SPOF, redundancia, degradacion, circuit breaker, bulkhead, radio de impacto, caos
---

## 🧠 Concepto

Alta disponibilidad no es "que no falle nada". Es que **el sistema siga sirviendo aunque algo falle**.

El cambio de mentalidad: en vez de intentar evitar los fallos —imposible—, se diseña para que ocurran sin tumbar el servicio.

```
❌ "¿Cómo evito que la base se caiga?"
✅ "¿Qué hace el sistema cuando la base se cae?"
```

## 🎯 Puntos únicos de falla

El primer ejercicio de cualquier diseño: **para cada componente, preguntá qué pasa si desaparece ahora**.

| Componente | Si se cae | ¿Es un punto único? |
|---|---|---|
| Una instancia de la app | otras siguen | ❌ si hay ≥ 2 |
| Base primaria | sin escrituras | ⚠️ mitigado con failover |
| Redis | más lento | ❌ si el caché es opcional |
| Broker de mensajes | eventos en cola | ⚠️ Outbox los retiene |
| Balanceador | **todo cae** | ✅ salvo redundancia |
| Servicio de identidad | nadie entra | ✅ |
| Una zona de disponibilidad | depende del despliegue | ✅ si todo está en una |

> [!danger] El punto único más común es el despliegue en una sola zona
> Tres réplicas en la misma zona no toleran nada: la zona cae y se van las tres. Distribuir entre zonas suele ser gratis dentro de la región, y es la diferencia entre tener y no tener alta disponibilidad.

## 🔢 Redundancia

| Modelo | Significa |
|---|---|
| **N** | justo la capacidad necesaria: sin tolerancia |
| **N+1** | uno de más: tolera una caída |
| **N+2** | dos de más: tolera una caída durante un mantenimiento |
| **2N** | el doble: tolera perder la mitad |

```
Necesito 4 instancias para el pico
→ desplegar 6 (N+2), repartidas en 3 zonas
→ si cae una zona (2 instancias), quedan 4: aguanta el pico ✅
```

> [!warning] Redundancia con dependencias compartidas no es redundancia
> Diez instancias de la aplicación contra **una** base son diez formas de fallar por lo mismo. La redundancia tiene que cubrir la cadena completa, o al menos declarar explícitamente dónde queda el punto único.

## 🪂 Degradación elegante

Cuando algo falla, servir menos es mejor que no servir nada.

```java
public PerfilCompleto obtenerPerfil(UUID clienteId) {
    var basico = clienteService.obtener(clienteId);       // crítico: si falla, falla todo

    var saldo = intentar(() -> cuentasService.saldo(clienteId))
        .orElse(Saldo.noDisponible());                    // degradable
    var recomendaciones = intentar(() -> recoService.para(clienteId))
        .orElse(List.of());                               // degradable

    return new PerfilCompleto(basico, saldo, recomendaciones);
}
```

> [!tip] Clasificá cada dependencia como crítica o degradable
> Es el ejercicio que más mejora la disponibilidad real, y casi nunca se hace explícitamente. Si el servicio de recomendaciones se cae, la pantalla de inicio debería mostrarse sin recomendaciones — no dar error 500.

| Nivel | Comportamiento |
|---|---|
| Completo | todo funciona |
| Degradado | funciones secundarias desactivadas |
| Solo lectura | se consulta, no se opera |
| Mantenimiento | mensaje claro, sin errores crípticos |

```java
// Interruptor de emergencia
if (features.activo("modo-solo-lectura")) {
    throw new ServicioEnModoLecturaException();
}
```

> [!tip] El modo solo lectura salva incidentes
> Si la base primaria se cae pero las réplicas están bien, el sistema puede seguir mostrando saldos y movimientos mientras no acepta operaciones. Para el usuario es muchísimo mejor que una pantalla de error.

## 🔌 Los cuatro patrones de resiliencia

```java
@CircuitBreaker(name = "cuentas", fallbackMethod = "respaldo")
@Retry(name = "cuentas")
@TimeLimiter(name = "cuentas")
@Bulkhead(name = "cuentas")
public CompletableFuture<Saldo> obtenerSaldo(UUID id) { ... }
```

| Patrón | Protege de | Sin él |
|---|---|---|
| **Timeout** | esperas indefinidas | hilos bloqueados, pool agotado |
| **Retry** | fallos transitorios | errores que se recuperaban solos |
| **Circuit breaker** | golpear un servicio caído | fallo en cascada |
| **Bulkhead** | que una dependencia consuma todo | un servicio lento tumba todo |

> [!danger] El timeout es el más importante y el que más falta
> Sin timeout, un servicio lento bloquea hilos indefinidamente. Con suficiente tráfico, el pool se agota y tu servicio **deja de responder aunque esté sano**. Es la causa más común de fallo en cascada.

```yaml
resilience4j:
  circuitbreaker:
    instances:
      cuentas:
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 3
  timelimiter:
    instances:
      cuentas: { timeoutDuration: 3s }
  bulkhead:
    instances:
      cuentas: { maxConcurrentCalls: 20 }
```

### El presupuesto de reintentos

```java
// ❌ Reintentos sin control amplifican la carga
// 3 servicios en cadena × 3 reintentos cada uno = 27 peticiones por una del usuario
```

> [!danger] Los reintentos se multiplican en cadena
> Si A reintenta 3 veces a B, y B reintenta 3 veces a C, una petición del usuario genera 9 llamadas a C. Con tres saltos son 27. **Justo cuando el sistema está en problemas, los reintentos lo empeoran.**
>
> Mitigaciones: reintentar solo en el borde, presupuesto de reintentos (máximo 10% del tráfico), y siempre con jitter.

```java
// Presupuesto: si ya se está reintentando mucho, no reintentar más
if (presupuestoDeReintentos.disponible()) {
    return conReintento(operacion);
}
return sinReintento(operacion);
```

## 💥 Radio de impacto

Cuánto se rompe cuando algo falla.

| Técnica | Cómo acota |
|---|---|
| **Compartimentos (cells)** | grupos de clientes en infraestructuras separadas |
| **Bulkhead** | pools separados por dependencia |
| **Despliegue progresivo** | 1% de usuarios primero |
| **Feature flags** | apagar una función sin desplegar |
| **Zonas** | un fallo de zona afecta a un tercio |

```
Sin compartimentos:  1 bug → 100% de clientes afectados
Con 4 compartimentos: 1 bug → 25% de clientes afectados
```

> [!tip] El despliegue progresivo es la red de seguridad más barata
> Desplegar al 1% de los usuarios, mirar las métricas 15 minutos y recién ahí continuar convierte un incidente masivo en uno acotado. Cuesta configurar una vez y paga en cada despliegue.

## 🩺 Health checks

```java
@Component
public class BaseDatosReadiness implements HealthIndicator {
    @Override public Health health() {
        return conexionOk() ? Health.up().build()
                            : Health.down().withDetail("motivo", "sin conexión").build();
    }
}
```

| Sonda | Contenido | Si falla |
|---|---|---|
| **Liveness** | ¿el proceso responde? | se reinicia el contenedor |
| **Readiness** | ¿puede atender? (incluye dependencias) | se saca del balanceador |

> [!danger] La base de datos nunca va en el liveness
> Si Postgres se cae, el liveness falla en todas las instancias, el orquestador las reinicia todas, y al volver siguen sin base. **Reiniciar en bucle no arregla una dependencia caída** — y borra el estado que permitiría diagnosticarla.

## 🧨 Ingeniería del caos

```java
@Test
@Tag("caos")
void el_sistema_degrada_cuando_cae_una_dependencia_secundaria() {
    toxiproxy.cortar("recomendaciones");
    var r = api.get("/perfil");
    assertEquals(200, r.statusCode());                     // sigue respondiendo
    assertTrue(r.jsonPath().getList("recomendaciones").isEmpty());
}

@Test
@Tag("caos")
void una_dependencia_lenta_no_agota_el_pool_de_hilos() {
    toxiproxy.agregarLatencia("cuentas", Duration.ofSeconds(30));
    ejecutarEnParalelo(200, () -> api.get("/saldo"));
    assertTrue(api.get("/actuator/health").statusCode() == 200);   // sigue vivo
}

@Test
@Tag("caos")
void la_caida_de_una_zona_no_interrumpe_el_servicio() { ... }
```

> [!warning] Un plan de resiliencia sin ensayar es una hipótesis
> El circuit breaker configurado y nunca probado, el fallback que nadie ejecutó, la réplica que jamás se promovió: todos "funcionan" hasta el día del incidente.
> Con **Toxiproxy** sobre Testcontainers se inyectan latencia, cortes y pérdida de paquetes de forma reproducible, en CI.

## 📊 Medir la disponibilidad

```
Disponibilidad = tiempo útil / tiempo total
```

| SLA | Caída mensual |
|---|---|
| 99% | 7,2 h |
| 99,9% | 43 min |
| 99,95% | 21 min |
| 99,99% | 4,3 min |
| 99,999% | 26 s |

> [!danger] Los SLA de las dependencias se multiplican
> ```
> Cómputo 99,95% × Base 99,99% × Almacenamiento 99,9% = 99,84%  ≈ 70 min/mes
> ```
> **Cada dependencia en serie baja el techo.** La forma de subirlo es tener caminos alternativos: caché que responde si la base no está, degradación, reintentos.

### Presupuesto de error

```
SLO: 99,9% mensual → presupuesto: 43 minutos

Consumiste 35 min a mitad de mes
→ se frena el despliegue de funcionalidades nuevas
→ se prioriza confiabilidad hasta recuperar margen
```

> [!tip] El presupuesto de error convierte la confiabilidad en una decisión medible
> Reemplaza la discusión sobre si "el sistema anda bien" por un número. Y hace explícito que gastar presupuesto en features está bien **mientras quede presupuesto**.

## ✅ Lista de verificación

- [ ] ¿Hay al menos 2 instancias de cada servicio?
- [ ] ¿Están repartidas en varias zonas?
- [ ] ¿Cada dependencia está clasificada como crítica o degradable?
- [ ] ¿Toda llamada saliente tiene timeout?
- [ ] ¿Los timeouts decrecen hacia adentro?
- [ ] ¿Hay circuit breaker en las llamadas remotas?
- [ ] ¿Los reintentos tienen jitter y presupuesto?
- [ ] ¿Liveness y readiness están separados?
- [ ] ¿La base está solo en el readiness?
- [ ] ¿El despliegue es progresivo con posibilidad de vuelta atrás?
- [ ] ¿Hay interruptores para apagar funciones?
- [ ] ¿Se probó el comportamiento ante fallos?
- [ ] ¿Está definido el SLO con presupuesto de error?

## ⚠️ Errores Comunes

> [!warning] Los que producen caídas totales
> - **Todo en una zona.**
> - **Llamadas sin timeout**: hilos bloqueados y pool agotado.
> - **Reintentos sin presupuesto**: amplifican la carga en el peor momento.
> - **Base de datos en el liveness**: bucle de reinicios.
> - **Sin clasificar dependencias**: un servicio secundario tumba el principal.
> - **Sin circuit breaker**: se sigue golpeando un servicio caído.
> - **Sin probar los fallos**: los mecanismos existen y no funcionan.
> - **Prometer un SLA mayor** que el producto de las dependencias.
> - **Una sola instancia**: cada despliegue es una caída.

## 📌 Notas Rápidas
- La pregunta que ordena todo: **"¿qué hace el sistema cuando X se cae?"**, aplicada a cada componente. Es un ejercicio de treinta minutos que encuentra más problemas que cualquier revisión de código.
- Clasificar dependencias en críticas y degradables es lo que más sube la disponibilidad real por unidad de esfuerzo, y casi nadie lo hace de forma explícita.
- El timeout es el patrón más importante y el que más falta. Si tuviera que revisar una sola cosa en un sistema ajeno, sería si todas las llamadas salientes lo tienen.
- Se aplica en [[Fase 6 - Microservicios y SAGA]] y [[Fase 7 - Observabilidad y Despliegue]].

## 🔗 Relaciones
- **Módulo**: [[System Design]]
- **Anterior**: [[07 - Rate Limiting]]
- **Siguiente**: [[09 - Tradeoffs]]
- **Se conecta con**: [[05 - Circuit Breaker - Resilience4j]], [[14 - Multiregion y DR]], [[01 - Modelo Mental Cloud]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

