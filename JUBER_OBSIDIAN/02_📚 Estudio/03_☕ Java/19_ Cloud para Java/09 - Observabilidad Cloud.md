---
tags:
  - Java
  - Cloud
  - Observabilidad
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - 06 - Logs Observabilidad y Monitoreo
Palabras clave:
  - OpenTelemetry, Application Insights, CloudWatch, X-Ray, KQL, trazas, muestreo, SLO
---

## 🧠 Concepto

Observabilidad en la nube tiene una particularidad: **no podés conectarte al servidor**. No hay SSH, no hay `tail -f`, y la instancia que falló probablemente ya no existe.

Todo lo que vas a saber del incidente es lo que la aplicación emitió **antes** de morir. Por eso la instrumentación no es un extra: es la única fuente de información.

## 🔭 OpenTelemetry primero

> [!tip] Instrumentá con OpenTelemetry, no con el SDK del proveedor
> OTel es el estándar de la industria (CNCF) y lo soportan las dos nubes. Instrumentás una vez y cambiás el destino con configuración. Con el SDK propietario, migrar de nube significa reinstrumentar todo.

```dockerfile
# Agente de OpenTelemetry: instrumenta sin tocar el código
ADD https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent.jar /otel.jar
ENV JAVA_TOOL_OPTIONS="-javaagent:/otel.jar"
ENV OTEL_SERVICE_NAME="corebank-api"
ENV OTEL_EXPORTER_OTLP_ENDPOINT="http://colector:4317"
ENV OTEL_TRACES_SAMPLER="parentbased_traceidratio"
ENV OTEL_TRACES_SAMPLER_ARG="0.1"
```

El agente instrumenta automáticamente Spring MVC, JDBC, Kafka, los clientes HTTP y los pools de conexiones. Sin una línea de código.

| Destino | Cómo |
|---|---|
| Azure Monitor | agente de Application Insights, o exportador OTLP |
| AWS | ADOT (distribución de AWS para OpenTelemetry) → CloudWatch + X-Ray |
| Genérico | colector OTLP → Grafana, Datadog, lo que sea |

## 📝 Logs

```xml
<!-- logback-spring.xml -->
<configuration>
  <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
      <includeMdcKeyName>traceId</includeMdcKeyName>
      <includeMdcKeyName>spanId</includeMdcKeyName>
      <includeMdcKeyName>usuarioId</includeMdcKeyName>
      <customFields>{"service":"corebank-api","env":"${ENV}"}</customFields>
    </encoder>
  </appender>
  <root level="INFO"><appender-ref ref="JSON"/></root>
</configuration>
```

```java
log.atInfo()
   .addKeyValue("evento", "transferencia_completada")
   .addKeyValue("asientoId", asiento.id())
   .addKeyValue("monto", importe.monto())
   .addKeyValue("duracionMs", duracion.toMillis())
   .log("transferencia completada");
```

> [!danger] Escribí a stdout, nunca a archivo
> En un contenedor efímero, un archivo de log desaparece con la instancia. La plataforma recoge stdout y lo envía al sistema de logs. Escribir a disco es perder exactamente los logs del momento en que algo falló.

### Consultarlos

```kusto
// Azure — Log Analytics (KQL)
AppTraces
| where TimeGenerated > ago(1h)
| where Properties.evento == "transferencia_completada"
| where todouble(Properties.duracionMs) > 1000
| project TimeGenerated, Properties.asientoId, Properties.duracionMs
| order by TimeGenerated desc
```

```sql
-- AWS — CloudWatch Logs Insights
fields @timestamp, asientoId, duracionMs
| filter evento = "transferencia_completada" and duracionMs > 1000
| sort @timestamp desc
| limit 50
```

> [!tip] Esto es lo que compran los logs estructurados
> Con texto plano, "todas las transferencias de más de un segundo la semana pasada" requiere expresiones regulares frágiles o directamente no se puede. Con campos, es una consulta.

## 📊 Métricas

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, prometheus, metrics
  metrics:
    tags:
      application: corebank-api
      environment: ${ENV}
```

```java
@Service
public class TransferirService {

    private final Counter completadas;
    private final Timer duracion;

    public TransferirService(MeterRegistry registry) {
        this.completadas = Counter.builder("corebank.transferencias")
            .tag("resultado", "completada").register(registry);
        this.duracion = Timer.builder("corebank.transferencias.duracion")
            .publishPercentiles(0.5, 0.95, 0.99)
            .register(registry);
    }
}
```

> [!danger] Cuidado con la cardinalidad de las etiquetas
> ```java
> .tag("cuentaId", cuenta.id())     // ❌ un millón de series temporales
> .tag("moneda", moneda.name())      // ✅ tres valores posibles
> ```
> Cada combinación de etiquetas crea una serie temporal que se almacena y se factura. Una etiqueta de alta cardinalidad puede multiplicar el costo de monitoreo por mil, y en algunos servicios directamente hace fallar la ingesta.
> **Los identificadores van en logs y trazas, nunca en etiquetas de métricas.**

### Las que no pueden faltar

| Métrica | Por qué |
|---|---|
| Tasa de peticiones, errores y latencia (RED) | salud del servicio |
| `hikaricp.connections.pending` | pool a punto de agotarse |
| `jvm.memory.used` / `jvm.gc.pause` | salud de la JVM |
| Profundidad de colas y DLQ | atascos en el procesamiento |
| Métricas de negocio | transferencias, rechazos, outbox pendiente |

> [!tip] Las métricas de negocio detectan lo que las técnicas no ven
> "CPU al 40%, latencia normal, cero errores" y aun así **cero transferencias en la última hora**. Técnicamente todo bien, funcionalmente todo mal. Solo una métrica de negocio muestra eso.

## 🔗 Trazas

```java
// El agente instrumenta lo estándar; agregás lo específico del dominio
@WithSpan("validar-antifraude")
public Decision evaluar(@SpanAttribute("cuentaId") UUID cuentaId, Operacion op) {
    Span.current().setAttribute("puntaje", puntaje);
    return decision;
}
```

```
Traza: POST /api/v1/transferencias         [────────────── 340 ms ──────────────]
├── validar-antifraude                     [── 45 ms ──]
├── SELECT cuenta FOR UPDATE                       [─ 12 ms ─]
├── INSERT asiento                                        [─ 8 ms ─]
├── INSERT outbox                                            [─ 5 ms ─]
└── HTTP POST cuentas-service/reservas                          [──── 210 ms ────]
```

> [!tip] La traza responde "¿dónde se fue el tiempo?"
> Sin ella, una petición lenta es un misterio: ¿la base, la red, el servicio externo, el GC? La traza lo muestra sin discusión — y en un sistema distribuido esa pregunta es la más frecuente.

### Muestreo

```
100% de trazas = costo altísimo y volumen inmanejable
1-10% = representativo para latencia
```

> [!danger] Nunca muestrees los errores
> El muestreo por proporción descarta trazas al azar, incluidas las de los fallos que querés investigar. La configuración correcta es **muestreo bajo para lo normal y 100% para errores y peticiones lentas**.

```yaml
# Muestreo basado en la cola: se decide DESPUÉS de ver la traza completa
OTEL_TRACES_SAMPLER: parentbased_traceidratio
OTEL_TRACES_SAMPLER_ARG: 0.1
# El colector conserva el 100% de las trazas con error
```

> [!warning] Propagá el contexto en los mensajes asíncronos
> Una traza se corta al pasar por una cola si no viajan las cabeceras `traceparent`. El agente de OTel lo hace por vos en Kafka y en varios clientes; en integraciones propias hay que inyectarlo a mano.

## 🎯 Los cuatro números

```java
@Component
public class ContextoDeTraza extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res,
                                    FilterChain chain) throws Exception {
        try {
            MDC.put("traceId", Span.current().getSpanContext().getTraceId());
            MDC.put("usuarioId", usuarioActual().map(u -> u.id().toString()).orElse("anonimo"));
            res.setHeader("X-Trace-Id", MDC.get("traceId"));    // el cliente puede citarlo
            chain.doFilter(req, res);
        } finally {
            MDC.clear();          // ⚠️ imprescindible
        }
    }
}
```

> [!danger] Sin `MDC.clear()` se filtran datos entre usuarios
> Los hilos se reutilizan entre peticiones. Si no limpiás el contexto, el `usuarioId` de una petición aparece en los logs de la siguiente — que es una fuga de datos, no solo un log confuso.

## 🚨 Alertas

```yaml
- alert: TasaDeErrorAlta
  expr: |
    sum(rate(http_server_requests_seconds_count{status=~"5.."}[5m]))
    / sum(rate(http_server_requests_seconds_count[5m])) > 0.01
  for: 5m

- alert: OutboxAtascado
  expr: corebank_outbox_pendientes > 1000
  for: 5m

- alert: PoolAgotado
  expr: hikaricp_connections_pending > 5
  for: 2m
```

> [!warning] Alertá sobre síntomas, no sobre causas
> "CPU al 85%" no es un problema si nadie lo nota. "El 3% de las transferencias falla" sí lo es. Las alertas sobre causas producen ruido, el ruido produce fatiga, y la fatiga produce alertas silenciadas — justo antes del incidente real.

### SLO y presupuesto de error

```
SLO: 99,9% de las transferencias responden en menos de 1 s
→ presupuesto de error: 0,1% del mes ≈ 43 minutos

Si consumiste el 80% del presupuesto a mitad de mes, se frena el despliegue de
funcionalidades nuevas y se prioriza confiabilidad.
```

> [!tip] El presupuesto de error convierte la confiabilidad en una decisión medible
> En vez de discutir si "el sistema anda bien", hay un número. Y hace explícito el compromiso: gastar el presupuesto en features es válido mientras quede presupuesto.

## 💰 El costo

> [!danger] La observabilidad puede costar más que el cómputo
> Es una sorpresa habitual: logs en modo DEBUG en producción, métricas de alta cardinalidad y trazas al 100% generan facturas que superan lo que cuesta correr la aplicación.

| Control | Ahorro |
|---|---|
| `INFO` en producción, no `DEBUG` | mucho |
| Muestreo de trazas al 1-10% | mucho |
| Sin etiquetas de alta cardinalidad | mucho |
| Retención corta para logs verbosos | medio |
| Archivar logs viejos a almacenamiento frío | medio |

## 🧪 Verificaciones

```java
@Test
void todos_los_logs_incluyen_traceId() {
    api.get("/api/v1/cuentas/" + id);
    assertTrue(capturador.entradas().stream()
        .allMatch(e -> e.getMDCPropertyMap().containsKey("traceId")));
}

@Test
void ningun_log_contiene_datos_sensibles() {
    servicio.autenticar("usuario", "contraseña-secreta");
    assertFalse(capturador.contenido().contains("contraseña-secreta"));
}

@Test
void el_MDC_se_limpia_entre_peticiones() {
    api.conUsuario("A").get("/cuentas");
    api.sinUsuario().get("/health");
    assertFalse(ultimaEntrada().getMDCPropertyMap().containsKey("usuarioId"));
}

@Test
void las_metricas_de_negocio_se_exponen() {
    servicio.transferir(cmd);
    assertEquals(1.0, registry.counter("corebank.transferencias", "resultado", "completada").count());
}

@Test
void la_traza_cruza_el_limite_asincrono() { ... }
```

## ⚠️ Errores Comunes

> [!warning] Los que te dejan a ciegas en el incidente
> - **Logs a archivo** en un contenedor efímero.
> - **Logs en texto plano**: imposibles de consultar.
> - **Sin `MDC.clear()`**: fuga de contexto entre peticiones.
> - **Etiquetas de métricas de alta cardinalidad**: explosión de costo.
> - **Muestrear también los errores.**
> - **Sin propagación de contexto** en colas: la traza se corta.
> - **Alertar sobre causas** en vez de síntomas.
> - **`DEBUG` en producción**: costo y ruido.
> - **Sin métricas de negocio**: todo "verde" y nada funcionando.
> - **Loguear el request completo**: contraseñas y tarjetas en el log.

## 📌 Notas Rápidas
- La prueba de fuego: agarrá un `traceId` real y reconstruí la petición completa entre servicios. Si no podés, falta instrumentación.
- OpenTelemetry con el agente automático da el 80% del valor sin escribir código. Empezá por ahí y agregá instrumentación manual solo donde el dominio lo pida.
- El `X-Trace-Id` en la respuesta es un detalle chico con mucho retorno: el usuario que reclama puede citarlo y el soporte encuentra la petición exacta en segundos.
- Se aplica en [[Fase 7 - Observabilidad y Despliegue]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[08 - Almacenamiento de Objetos]]
- **Siguiente**: [[10 - Red y Exposicion]]
- **Se conecta con**: [[06 - Logs Observabilidad y Monitoreo]], [[08 - Distributed Tracing]], [[03 - Application Insights y Azure Monitor]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

