---
tags:
  - Java
  - Proyecto
  - Banca
  - corebank
  - DevOps
Creado: 2026-09-08
Relacionado:
  - Proyectos Java
  - DevOps con Java
  - Cloud para Java
Palabras clave:
  - observabilidad, logs estructurados, Micrometer, OpenTelemetry, CI/CD, Docker, cloud
Estado:
  - Pendiente
Nivel: Avanzado
Prerequisitos: Fase 6, DevOps con Java, Cloud para Java
---

# Fase 7 · Observabilidad y despliegue

## 🎯 Objetivo

`corebank` funciona en tu máquina. En esta fase se vuelve **operable**: se puede desplegar sin intervención manual, y cuando algo falla a las 3 de la mañana se puede diagnosticar sin adivinar.

> [!info] La diferencia entre monitoreo y observabilidad
> **Monitoreo** responde preguntas que anticipaste: "¿está arriba?", "¿cuánta CPU usa?".
> **Observabilidad** te deja responder preguntas que **no** anticipaste: "¿por qué las transferencias del cliente X en soles tardan 4 segundos solo los martes?".
> Lo primero son dashboards. Lo segundo exige que la aplicación emita el contexto correcto.

## 🧠 Los tres pilares

| Pilar | Responde | Herramienta |
|---|---|---|
| **Logs** | qué pasó exactamente | Logback JSON → Loki / CloudWatch |
| **Métricas** | cuánto y con qué frecuencia | Micrometer → Prometheus → Grafana |
| **Trazas** | por dónde pasó y dónde se demoró | OpenTelemetry → Tempo / Jaeger |

Lo que los une es el **trace id**: un identificador que nace en el gateway y viaja por todos los servicios. Con él, un log te lleva a la traza y la traza a la métrica.

## 📝 Logs estructurados

```java
// ❌ imposible de consultar
log.info("Transferencia de " + monto + " desde " + origen + " a " + destino);

// ✅ campos consultables
log.atInfo()
   .addKeyValue("evento", "transferencia_completada")
   .addKeyValue("asientoId", asiento.id())
   .addKeyValue("cuentaOrigen", origen.id())
   .addKeyValue("monto", importe.monto())
   .addKeyValue("moneda", importe.moneda())
   .addKeyValue("duracionMs", duracion.toMillis())
   .log("transferencia completada");
```

```json
{
  "timestamp": "2026-09-08T14:23:11.482Z",
  "level": "INFO",
  "service": "transferencias-service",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "spanId": "00f067aa0ba902b7",
  "usuarioId": "9c1e...",
  "evento": "transferencia_completada",
  "asientoId": "7d3a...",
  "monto": 500.00,
  "moneda": "PEN",
  "duracionMs": 143,
  "message": "transferencia completada"
}
```

Con esto podés preguntar *"todas las transferencias de más de S/ 10.000 que tardaron más de 1 segundo la semana pasada"*. Con logs de texto plano, no.

### El contexto viaja solo (MDC)

```java
@Component
public class FiltroContexto extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res,
                                    FilterChain chain) throws Exception {
        try {
            MDC.put("traceId", Span.current().getSpanContext().getTraceId());
            MDC.put("usuarioId", usuarioActual().map(u -> u.id().toString()).orElse("anonimo"));
            MDC.put("ip", ipReal(req));
            res.setHeader("X-Trace-Id", MDC.get("traceId"));   // el cliente puede citarlo
            chain.doFilter(req, res);
        } finally {
            MDC.clear();     // ⚠️ imprescindible: los hilos se reutilizan
        }
    }
}
```

> [!danger] Nunca loguees datos sensibles
> Contraseñas, tokens, números de tarjeta, documentos de identidad. Los logs se copian, se exportan y se indexan en servicios de terceros. Una tarjeta en un log es un incidente de PCI-DSS.
> Enmascará en el propio serializador, no confiando en que cada desarrollador se acuerde.

```java
public record TarjetaEnmascarada(String pan) {
    @Override public String toString() {
        return "****" + pan.substring(pan.length() - 4);
    }
}
```

## 📊 Métricas

```java
@Service
public class TransferirService {

    private final Counter completadas;
    private final Counter rechazadas;
    private final Timer duracion;

    public TransferirService(MeterRegistry registry) {
        this.completadas = Counter.builder("corebank.transferencias")
            .tag("resultado", "completada").register(registry);
        this.rechazadas  = Counter.builder("corebank.transferencias")
            .tag("resultado", "rechazada").register(registry);
        this.duracion    = Timer.builder("corebank.transferencias.duracion")
            .publishPercentiles(0.5, 0.95, 0.99)
            .register(registry);
    }
}
```

### Las que hay que tener

| Métrica | Tipo | Por qué |
|---|---|---|
| `corebank.transferencias{resultado}` | contador | volumen y tasa de rechazo |
| `corebank.transferencias.duracion` | timer p50/p95/p99 | latencia real |
| `corebank.outbox.pendientes` | gauge | si sube, el publicador se atascó |
| `corebank.saga.en_curso{estado}` | gauge | sagas colgadas |
| `corebank.dlt.profundidad` | gauge | eventos muertos sin atender |
| `hikaricp.connections.active` | gauge | pool a punto de agotarse |
| `jvm.gc.pause` | timer | pausas de GC |
| `http.server.requests{status}` | timer | tasa de error por endpoint |

> [!tip] Medí el p99, no el promedio
> Con 1000 peticiones de 10 ms y 10 de 5 segundos, el promedio da 60 ms y parece sano. El p99 muestra los 5 segundos. **El promedio esconde exactamente lo que te importa.**

### Alertas útiles

```yaml
- alert: OutboxAtascado
  expr: corebank_outbox_pendientes > 1000
  for: 5m
  annotations:
    summary: "Outbox con {{ $value }} eventos sin publicar"

- alert: SagasColgadas
  expr: corebank_saga_en_curso{estado!~"COMPLETADA|FALLIDA"} > 10
  for: 10m

- alert: TasaDeErrorAlta
  expr: |
    sum(rate(http_server_requests_seconds_count{status=~"5.."}[5m]))
    / sum(rate(http_server_requests_seconds_count[5m])) > 0.01
  for: 5m
```

> [!warning] Alertá sobre síntomas, no sobre causas
> "CPU al 80%" no es un problema si nadie lo nota. "El 5% de las transferencias falla" sí lo es. Alertas sobre causas producen fatiga y terminan silenciadas.

## 🩺 Health checks

```java
@Component
public class OutboxHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        long pendientes = repo.contarPendientes();
        Instant masAntiguo = repo.fechaDelMasAntiguo();

        if (masAntiguo != null && masAntiguo.isBefore(Instant.now().minus(Duration.ofMinutes(5)))) {
            return Health.down()
                .withDetail("pendientes", pendientes)
                .withDetail("masAntiguo", masAntiguo)
                .build();
        }
        return Health.up().withDetail("pendientes", pendientes).build();
    }
}
```

| Endpoint | Para qué | Si falla |
|---|---|---|
| `/actuator/health/liveness` | ¿el proceso está vivo? | se reinicia el contenedor |
| `/actuator/health/readiness` | ¿puede recibir tráfico? | se saca del balanceador |

> [!danger] No pongas la base en el liveness
> Si Postgres se cae, el liveness falla, Kubernetes reinicia **todos** los pods, y al volver siguen sin base. Reiniciar en bucle no arregla una dependencia caída. La base va en el **readiness**.

## 🐳 Dockerfile

```dockerfile
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline -B          # capa cacheada
COPY src ./src
RUN ./mvnw clean package -DskipTests -B

FROM eclipse-temurin:21-jre-alpine
RUN addgroup -S app && adduser -S app -G app
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
USER app                                      # nunca root
EXPOSE 8080
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75 -XX:+UseG1GC -XX:+ExitOnOutOfMemoryError"
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

> [!tip] `MaxRAMPercentage` en vez de `-Xmx`
> Con `-Xmx` fijo, cambiar el límite de memoria del contenedor requiere reconstruir la imagen. Con el porcentaje, la JVM se adapta sola al límite del cgroup.
> Y `ExitOnOutOfMemoryError` es clave: sin él, una JVM sin memoria queda viva pero inservible, y el orquestador no la reemplaza.

## ⚙️ CI/CD

```yaml
name: ci
on: [push, pull_request]

jobs:
  verificar:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { java-version: '21', distribution: 'temurin', cache: maven }

      - name: Tests de dominio
        run: ./mvnw test -Dgroups=dominio

      - name: Reglas de arquitectura
        run: ./mvnw test -Dtest=ReglasDeArquitectura

      - name: Integración (Testcontainers)
        run: ./mvnw verify -Pintegracion

      - name: Contract tests
        run: ./mvnw pact:verify

      - name: Mutation testing
        run: ./mvnw pitest:mutationCoverage

      - name: Análisis de dependencias
        run: ./mvnw dependency-check:check     # CVEs conocidos
```

El orden importa: **lo más rápido y lo que más falla, primero**. Los tests de dominio corren en segundos; no tiene sentido esperar los de integración para descubrir un error de lógica.

## ☁️ Despliegue

Los conceptos son los mismos en las dos nubes — ver [[14 - Equivalencias Azure AWS]]:

| Necesidad | Azure | AWS |
|---|---|---|
| Correr contenedores | Container Apps | ECS Fargate |
| Registro de imágenes | ACR | ECR |
| Secretos | Key Vault | Secrets Manager |
| Identidad sin credenciales | Managed Identity | IAM Role for Task |
| Base de datos | PostgreSQL Flexible Server | RDS PostgreSQL |
| Observabilidad | Application Insights | CloudWatch + X-Ray |

```java
// El código NO sabe en qué nube corre: lee variables de entorno
@ConfigurationProperties(prefix = "corebank.db")
public record ConfigBaseDatos(String url, String usuario, String password) {}
```

> [!danger] Ningún secreto en la imagen, el repo ni el `application.yml`
> Los secretos se inyectan en tiempo de ejecución desde el almacén de la nube, y la app se autentica con identidad gestionada — sin contraseña que rotar ni filtrar. Es el punto entero de Managed Identity e IAM Roles.

## ✅ Criterios de aceptación

- [ ] Todos los logs son JSON estructurado con `traceId`, `spanId` y `service`.
- [ ] El `traceId` viaja entre servicios y vuelve al cliente en `X-Trace-Id`.
- [ ] Ningún dato sensible aparece en los logs (verificado con un test).
- [ ] Métricas de negocio expuestas en `/actuator/prometheus`.
- [ ] Dashboard de Grafana con volumen, latencia p50/p95/p99 y tasa de error.
- [ ] Alertas configuradas para outbox atascado, sagas colgadas y tasa de error.
- [ ] `liveness` y `readiness` separados; la base solo en readiness.
- [ ] Imagen Docker multi-stage, con usuario no root y menos de 250 MB.
- [ ] Pipeline que corre tests, arquitectura, contratos, mutación y CVEs.
- [ ] Desplegado en una nube, accesible por HTTPS.
- [ ] Cero secretos en el repositorio o en la imagen.
- [ ] Podés reconstruir una transferencia completa a partir de su `traceId`.

## ⚠️ Trampas de esta fase

> [!warning] Las que se pagan en producción
> - **Loguear en texto plano.** Imposible de consultar cuando más lo necesitás.
> - **Olvidar `MDC.clear()`.** Los hilos se reutilizan y el `usuarioId` de una petición se filtra a la siguiente. Es una fuga de datos entre usuarios.
> - **Base de datos en el liveness.** Convierte una caída de base en un bucle de reinicios.
> - **Loguear el request completo.** Ahí van tokens y contraseñas.
> - **Alertar sobre todo.** Cien alertas por día y nadie mira ninguna.
> - **`latest` como tag de imagen.** No sabés qué está desplegado ni podés volver atrás.

## 🔁 Retrospectiva

- Tomá un `traceId` real: ¿podés reconstruir la petición completa entre servicios?
- ¿Qué métrica te avisaría primero de que el outbox se atascó?
- ¿Cuánto tarda el pipeline? ¿Qué paso es el cuello de botella?
- Si te llaman a las 3am por "transferencias lentas", ¿cuál es tu primer paso concreto?
- ¿Qué pasa si Postgres se cae 5 minutos? ¿Y si es Kafka?

## 🔗 Relaciones
- **Anterior**: [[Fase 6 - Microservicios y SAGA]]
- **Siguiente**: [[Fase 8 - Batch y Cierre de Dia]]
- **Teoría que aplica**: [[06 - Logs Observabilidad y Monitoreo]], [[08 - Distributed Tracing]], [[03 - CI CD para Java]], [[Cloud para Java]], [[09 - Observabilidad Cloud]]
- **Índice**: [[Proyectos Java]]

## 📌 Notas Rápidas
- La pregunta de las 3am es el mejor test de observabilidad: si la respuesta empieza con "me conecto por SSH y miro los logs", falta trabajo.
- El costo de observabilidad es real: logs y trazas de alto volumen se facturan. Se muestrean las trazas (1-10%) pero **nunca** los errores: esos se guardan siempre.
- Con esto, `corebank` ya es un sistema desplegable y operable de punta a punta — que es exactamente la evidencia que buscabas del laboratorio.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

