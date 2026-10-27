---
tags:
  - Postulaciones
  - Observabilidad
  - OpenTelemetry
  - Azure
Creado: 2026-08-30
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Observabilidad: logs, métricas y trazas

## La respuesta corta sobre MongoDB

No agregues MongoDB para guardar logs solo porque acepta JSON. Un sistema de observabilidad necesita ingestión, esquema consistente, correlación, retención, consultas temporales, dashboards y alertas. En Azure, aprende primero **Application Insights + Log Analytics**, instrumentados con **OpenTelemetry**.

MongoDB se estudia como base documental cuando el dominio lo necesita, no como reemplazo improvisado de una plataforma de observabilidad.

## Las tres señales esenciales

| Señal | Pregunta que responde | Ejemplo |
|---|---|---|
| Logs | ¿Qué evento ocurrió? | fallo de validación o excepción |
| Métricas | ¿Cuánto y con qué tendencia? | latencia p95, error rate, CPU |
| Trazas | ¿Por dónde pasó la solicitud? | Angular → API → DB → Service Bus |

OpenTelemetry normaliza y correlaciona estas señales; el backend puede ser Azure Monitor u otra plataforma compatible. Ver [señales de OpenTelemetry](https://opentelemetry.io/docs/concepts/signals/).

## Qué implementar en .NET

- `ILogger` con message templates y campos estables.
- Niveles correctos: Debug, Information, Warning, Error, Critical.
- `traceId`, `spanId`, `correlationId`, `tenantId` cuando corresponda.
- `ActivitySource` para spans propios.
- `Meter` para métricas de negocio y técnicas.
- Instrumentación automática de ASP.NET Core, HTTP y base de datos.
- Health checks separados para liveness y readiness.
- Redacción de tokens, contraseñas y datos personales.

Azure Monitor ofrece una distribución de OpenTelemetry para recopilar trazas, métricas, logs y excepciones en Application Insights: [OpenTelemetry en Application Insights](https://learn.microsoft.com/azure/azure-monitor/app/opentelemetry-enable?tabs=aspnetcore).

## Qué observar en Angular

- errores no controlados;
- fallos y latencia HTTP;
- navegación y tiempos relevantes;
- versión del frontend;
- Core Web Vitals si el producto lo requiere;
- contexto suficiente para reproducir, sin capturar datos sensibles.

## Dashboard mínimo

- Requests por minuto.
- Error rate.
- Latencia p50, p95 y p99.
- Dependencias lentas o fallidas.
- Excepciones por versión.
- Estado de health checks.
- Una métrica de negocio, por ejemplo registros o cursos completados.

## Alerta mínima

- error rate sobre umbral durante varios minutos;
- p95 por encima del objetivo;
- health check fallando;
- cola creciendo sin consumo;
- base de datos o dependencia con fallos sostenidos.

## Laboratorio de entrevista

1. Provocar una excepción controlada.
2. Encontrarla en Application Insights.
3. Seguir el trace hasta la dependencia que falló.
4. Crear una consulta KQL.
5. Definir una alerta.
6. Explicar cómo reducir ruido y costo mediante niveles, sampling y retención.

## Criterio de salida

- [ ] Puedo seguir una petición de extremo a extremo.
- [ ] Distingo logging de observabilidad.
- [ ] No registro secretos ni información sensible.
- [ ] Puedo explicar SLI, SLO y una alerta accionable.

Repasar: [[Logging tracing metrics OpenTelemetry]], [[04 - Application Insights]], [[05 - Distributed Tracing OpenTelemetry]], [[Playbook de Incidentes dotNET]].

