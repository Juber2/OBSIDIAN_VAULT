---
tags:
  - Azure
  - Observabilidad
  - ApplicationInsights
Creado: 2026-08-30
Relacionado:
  - Azure
  - Laboratorios
---

# Application Insights y Azure Monitor

> Pregunta literal de la vacante de Entelgy: **"¿Qué observas en Application Insights durante un incidente?"**. Observabilidad aparece en 4 de 14 vacantes del radar. La nota [[06 - SRE APM y IaC]] cubre la teoría (SLI/SLO, los tres pilares); esta cubre cómo se usa.

## Los tres pilares, en concreto

| Pilar | Responde a | Dónde vive en Azure |
|---|---|---|
| **Logs** | ¿Qué pasó exactamente en este caso? | Tablas `traces`, `exceptions` |
| **Métricas** | ¿Cómo se comporta el sistema en agregado? | Tablas `customMetrics`, métricas de plataforma |
| **Trazas** | ¿Por dónde pasó esta petición entre servicios? | Tablas `requests`, `dependencies` |

Los tres se consultan con **KQL** (Kusto Query Language) desde Application Insights o Log Analytics.

## Instrumentar la aplicación

### .NET

```csharp
builder.Services.AddApplicationInsightsTelemetry();
```

Con esa línea ya obtenés automáticamente: requests HTTP entrantes, dependencias salientes (HTTP, SQL, Service Bus), excepciones no manejadas y correlación entre servicios. **La instrumentación automática cubre el 80% — lo que agregás a mano es el contexto de negocio.**

```csharp
_logger.LogInformation("Orden {OrdenId} procesada para cliente {ClienteId} en {Monto}",
    orden.Id, orden.ClienteId, orden.Monto);
```

Usar plantillas con placeholders (no interpolación de strings) es lo que convierte un log en **estructurado**: `OrdenId` queda como campo consultable, no como texto dentro de un mensaje.

```csharp
// ❌ No consultable: el ID queda enterrado en el texto
_logger.LogInformation($"Orden {orden.Id} procesada");

// ✅ Consultable: customDimensions.OrdenId
_logger.LogInformation("Orden {OrdenId} procesada", orden.Id);
```

### Java/Spring

Application Insights para Java funciona como agente, sin tocar el código:

```bash
java -javaagent:applicationinsights-agent.jar -jar mi-api.jar
```

Para telemetría propia, Micrometer + OpenTelemetry es el camino estándar y portable (sirve igual si mañana el cliente usa Grafana en vez de Azure).

## KQL — las consultas que realmente usás en un incidente

Esta es la parte que separa "sé qué es App Insights" de "sé usarlo".

### 1. ¿Qué está fallando ahora?

```kusto
requests
| where timestamp > ago(1h)
| where success == false
| summarize fallos = count() by name, resultCode
| order by fallos desc
```

### 2. ¿Se degradó la latencia? (percentiles, no promedio)

```kusto
requests
| where timestamp > ago(6h)
| summarize p50 = percentile(duration, 50),
            p95 = percentile(duration, 95),
            p99 = percentile(duration, 99)
          by bin(timestamp, 5m)
| render timechart
```

> El promedio miente: con 99 peticiones de 100ms y una de 10 segundos, el promedio queda en ~200ms y parece sano. **El p95/p99 es lo que sienten los usuarios afectados.** Responder "miro el promedio de latencia" en una entrevista es una señal negativa.

### 3. ¿La culpa es mía o de una dependencia?

```kusto
dependencies
| where timestamp > ago(1h)
| summarize llamadas = count(),
            fallidas = countif(success == false),
            p95 = percentile(duration, 95)
          by target, type
| order by p95 desc
```

Esto responde la pregunta más importante del primer minuto de un incidente: si el p95 alto está en `dependencies` hacia SQL, el problema es la base, no tu código.

### 4. Seguir una petición concreta entre servicios

```kusto
union requests, dependencies, traces, exceptions
| where operation_Id == "a1b2c3d4e5f6"
| order by timestamp asc
| project timestamp, itemType, name, success, duration, message
```

`operation_Id` es el **correlation ID**: se propaga automáticamente entre servicios instrumentados. Esta única consulta reconstruye el recorrido completo de una petición, atravesando API → Service Bus → Function → base de datos.

### 5. Excepciones agrupadas por causa

```kusto
exceptions
| where timestamp > ago(24h)
| summarize count() by type, outerMessage
| order by count_ desc
```

## Correlation ID: por qué importa

```
Cliente ──► API (operation_Id: abc123)
              │  propaga el mismo abc123 en el header traceparent
              ▼
           Service Bus (mensaje lleva abc123)
              ▼
           Azure Function (operation_Id: abc123)
              ▼
           Azure SQL (dependency bajo abc123)
```

Sin correlación tenés cinco sistemas con logs separados y ninguna forma de saber cuáles pertenecen a la misma petición. Con correlación, una consulta reconstruye todo. En arquitectura de microservicios **esto no es opcional**: es la diferencia entre diagnosticar en minutos u horas.

El estándar es W3C Trace Context (header `traceparent`), soportado por App Insights y OpenTelemetry — no es específico de Azure.

## Alertas accionables

```bash
az monitor metrics alert create \
  --name "latencia-p95-alta" \
  --resource-group mi-rg \
  --scopes /subscriptions/<sub>/resourceGroups/mi-rg/providers/Microsoft.Web/sites/mi-api \
  --condition "avg HttpResponseTime > 2" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action <action-group-id>
```

Una alerta es accionable cuando cumple tres cosas:

1. **Indica un problema real para el usuario**, no una métrica técnica curiosa (CPU al 80% no es un problema si la latencia está bien).
2. **Alguien puede hacer algo al recibirla** — si la respuesta es "esperar a ver si se arregla", no debería ser una alerta.
3. **No se repite tanto que se ignora.** Una alerta que suena 40 veces por día deja de leerse; la fatiga de alertas es un problema operativo, no de configuración.

> Buena respuesta de entrevista sobre qué alertar: **tasa de error, latencia p95, saturación de recursos y profundidad de colas** — no "CPU alta" a secas.

## Qué mirar en un incidente, en orden

Este es el guion concreto para responder la pregunta de Entelgy:

1. **¿Cuántos usuarios afecta?** — tasa de fallos en `requests`, para dimensionar la gravedad.
2. **¿Desde cuándo?** — timechart para correlacionar con un deploy o un cambio de configuración.
3. **¿Es mío o de una dependencia?** — comparar `requests` contra `dependencies`.
4. **¿Qué excepción concreta?** — `exceptions` agrupadas por tipo.
5. **Un caso completo** — seguir un `operation_Id` de punta a punta.
6. **Mitigar antes de entender del todo** — rollback o swap de slot restaura el servicio; el análisis de causa raíz viene después, no durante.

El punto 6 es el que más diferencia a un perfil senior: **mitigar y diagnosticar son dos fases distintas**, y el usuario no puede esperar a que entiendas la causa.

## Costo: sampling

App Insights cobra por volumen ingerido. En una API con tráfico alto, la telemetría puede costar más que el cómputo.

```csharp
builder.Services.Configure<TelemetryConfiguration>(config =>
{
    // Conservar el 100% de errores, muestrear el resto
    config.DefaultTelemetrySink.TelemetryProcessorChainBuilder
        .UseAdaptiveSampling(maxTelemetryItemsPerSecond: 5, excludedTypes: "Exception");
});
```

Excluir excepciones del sampling es la decisión correcta: nunca querés perder el error justo que estabas buscando.

## Errores comunes

- Loguear con interpolación de strings — el dato deja de ser consultable.
- Mirar promedios en vez de percentiles.
- Loguear tokens, passwords o datos personales (ver [[03 - Seguridad Backend Java]], A09 de OWASP).
- Alta cardinalidad: usar un ID de usuario como dimensión de métrica genera millones de series y dispara el costo.
- Alertar sobre métricas técnicas que no afectan al usuario, hasta que nadie lee las alertas.
- Instrumentar sin propagar correlación entre servicios: tenés datos, pero no podés unirlos.

## Checklist

- [ ] Sé escribir un log estructurado y explicar por qué la interpolación lo arruina.
- [ ] Puedo escribir de memoria una consulta KQL de tasa de error y otra de p95.
- [ ] Sé distinguir si un problema es de mi servicio o de una dependencia.
- [ ] Puedo explicar qué es `operation_Id` y cómo se propaga entre servicios.
- [ ] Puedo decir qué hace accionable a una alerta y dar tres ejemplos correctos.
- [ ] Tengo el guion de 6 pasos para un incidente y entiendo por qué mitigar va antes que diagnosticar.
- [ ] Sé qué es cardinalidad y por qué encarece la telemetría.
