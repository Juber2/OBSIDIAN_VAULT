---
tags:
  - Java
  - Spring Boot
  - Batch
Creado: 2026-08-12
Relacionado:
  - Spring Boot
---

# Jobs Programados y Spring Batch

Todo backend real termina necesitando tareas que corren solas: limpiar datos viejos, generar reportes, reconciliar estados, enviar recordatorios. Spring ofrece tres niveles según la complejidad.

## Nivel 1: `@Scheduled` — tareas simples periódicas

```java
@Configuration
@EnableScheduling
public class SchedulingConfig {}
```

```java
@Component
@Slf4j
public class LimpiezaTareas {

    // Cada 1 hora exacta (fixedRate: mide desde el INICIO de la ejecución anterior)
    @Scheduled(fixedRate = 3_600_000)
    public void limpiarSesionesExpiradas() {
        log.info("Limpiando sesiones expiradas");
    }

    // Cada 30 minutos DESPUÉS de que termina la ejecución anterior (fixedDelay)
    @Scheduled(fixedDelay = 1_800_000)
    public void sincronizarInventario() {
        log.info("Sincronizando inventario");
    }

    // Cron — control total sobre el horario
    @Scheduled(cron = "0 0 3 * * *")   // todos los días a las 3:00 AM
    public void generarReporteDiario() {
        log.info("Generando reporte diario");
    }

    // Con zona horaria explícita (importante en apps multi-región)
    @Scheduled(cron = "0 0 9 * * MON-FRI", zone = "America/Lima")
    public void enviarRecordatoriosLaborales() {
        log.info("Enviando recordatorios");
    }
}
```

### `fixedRate` vs `fixedDelay` (pregunta común de entrevista)

```
fixedRate:  |--- tarea 1 (2s) ---|          |--- tarea 2 (2s) ---|
            0s                   5s (fixedRate=5s)              10s
            → la siguiente arranca 5s después del INICIO de la anterior

fixedDelay: |--- tarea 1 (2s) ---|-- espera 5s --|--- tarea 2 ---|
            0s                   2s              7s
            → la siguiente arranca 5s después de que TERMINÓ la anterior
```

`fixedDelay` es más seguro cuando la tarea puede tardar variablemente — evita que dos ejecuciones se superpongan si una tarda más de lo esperado.

### Configurar el pool de threads (por default corre en 1 solo thread)

```properties
spring.task.scheduling.pool.size=5
```

Sin esto, todos los `@Scheduled` compiten por **un solo thread** — si una tarea se cuelga, bloquea a todas las demás.

## Nivel 2: `TaskScheduler` programático — cuando el horario es dinámico

```java
@Service
@RequiredArgsConstructor
public class RecordatorioService {

    private final TaskScheduler taskScheduler;

    public void programarRecordatorio(LocalDateTime fecha, Runnable accion) {
        Instant instante = fecha.atZone(ZoneId.systemDefault()).toInstant();
        taskScheduler.schedule(accion, instante);
    }
}
```

Útil cuando el momento de ejecución depende de datos de negocio (ej. "recordar 24h antes de la fecha de vencimiento de cada pedido"), no un cron fijo conocido de antemano.

## Nivel 3: Spring Batch — procesamiento masivo estructurado

Cuando la tarea no es "ejecutar un método", sino **procesar grandes volúmenes de datos** (leer, transformar, escribir, con checkpoints y reintentos), `@Scheduled` con un método simple ya no alcanza — ahí entra Spring Batch.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

### Conceptos

| Concepto | Qué es |
|---|---|
| **Job** | El proceso batch completo |
| **Step** | Una fase del Job (puede haber varios en secuencia) |
| **ItemReader** | Lee los datos de origen (BD, archivo, API) |
| **ItemProcessor** | Transforma cada item leído |
| **ItemWriter** | Escribe el resultado (BD, archivo, otro sistema) |
| **Chunk** | Procesa de a N items por vez, con commit periódico (no todo en una sola transacción gigante) |

```java
@Configuration
public class ReporteBatchConfig {

    @Bean
    public Job generarReporteJob(JobRepository jobRepository, Step procesarPedidosStep) {
        return new JobBuilder("generarReporteJob", jobRepository)
            .start(procesarPedidosStep)
            .build();
    }

    @Bean
    public Step procesarPedidosStep(
            JobRepository jobRepository,
            PlatformTransactionManager transactionManager,
            ItemReader<Pedido> reader,
            ItemProcessor<Pedido, ReporteDTO> processor,
            ItemWriter<ReporteDTO> writer) {

        return new StepBuilder("procesarPedidosStep", jobRepository)
            .<Pedido, ReporteDTO>chunk(100, transactionManager)   // commit cada 100 items
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .faultTolerant()
            .retryLimit(3)
            .retry(TransientDataAccessException.class)   // reintentar en errores transitorios
            .skipLimit(10)
            .skip(ValidationException.class)              // saltar hasta 10 items inválidos
            .build();
    }

    @Bean
    public ItemReader<Pedido> pedidoReader(EntityManagerFactory emf) {
        return new JpaPagingItemReaderBuilder<Pedido>()
            .name("pedidoReader")
            .entityManagerFactory(emf)
            .queryString("SELECT p FROM Pedido p WHERE p.creadoEn >= :desde")
            .parameterValues(Map.of("desde", LocalDate.now().minusDays(1)))
            .pageSize(100)
            .build();
    }

    @Bean
    public ItemProcessor<Pedido, ReporteDTO> pedidoProcessor() {
        return pedido -> new ReporteDTO(pedido.getId(), pedido.getTotal());
    }

    @Bean
    public ItemWriter<ReporteDTO> reporteWriter(ReporteRepository repo) {
        return items -> repo.saveAll(items);
    }
}
```

### Disparar el Job

```java
@Component
@RequiredArgsConstructor
public class ReporteJobLauncher {

    private final JobLauncher jobLauncher;
    private final Job generarReporteJob;

    @Scheduled(cron = "0 0 2 * * *")
    public void ejecutar() throws Exception {
        JobParameters params = new JobParametersBuilder()
            .addLong("timestamp", System.currentTimeMillis())   // parámetro único obliga a re-ejecutar
            .toJobParameters();
        jobLauncher.run(generarReporteJob, params);
    }
}
```

### Por qué Spring Batch y no solo un `for` con `@Scheduled`

- **Checkpoints/restart**: si el Job falla en el item 5000 de 10000, puede reanudar desde ahí, no desde cero.
- **Chunk processing**: no carga millones de registros en memoria de una vez ni hace un solo commit gigante.
- **Skip/Retry declarativo**: manejar errores fila por fila sin ensuciar la lógica de negocio con try/catch manual repetido.
- **JobRepository**: guarda el historial de ejecuciones — auditable, se puede consultar qué corrió, cuándo, con qué resultado.

## Cuándo NO usar Spring Batch

Para una tarea simple ("marcar como inactivos los usuarios sin login en 90 días" sobre unos miles de registros), `@Scheduled` + una query directa es más simple y suficiente. Spring Batch se justifica cuando el volumen es alto, la tarea es crítica (necesita reintentos/auditoría), o el procesamiento tiene múltiples fases.

## Quartz — alternativa cuando se necesita más control de scheduling

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

Quartz aporta lo que `@Scheduled` no tiene nativamente: persistir el schedule en base de datos (sobrevive a un reinicio y es consultable/modificable en runtime), y coordinación entre múltiples instancias para que **solo una** ejecute el job en un momento dado (evita que 3 réplicas del mismo Pod corran el mismo job 3 veces).

```java
@Bean
public JobDetail reporteJobDetail() {
    return JobBuilder.newJob(ReporteQuartzJob.class)
        .withIdentity("reporteJob")
        .storeDurably()
        .build();
}

@Bean
public Trigger reporteTrigger(JobDetail reporteJobDetail) {
    return TriggerBuilder.newTrigger()
        .forJob(reporteJobDetail)
        .withSchedule(CronScheduleBuilder.cronSchedule("0 0 3 * * ?"))
        .build();
}
```

## Scheduling en Kubernetes: alternativa nativa

En vez de resolver "solo una instancia debe correr el job" a nivel de aplicación (Quartz con lock distribuido), Kubernetes lo resuelve a nivel de plataforma con un **CronJob**:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: generar-reporte
spec:
  schedule: "0 3 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: reporte
              image: miapp/reporte-batch:1.0
          restartPolicy: OnFailure
```

Esto crea un Pod nuevo que ejecuta la tarea y termina, en vez de tener la lógica de scheduling viviendo dentro de una app que corre 24/7 — más alineado con el modelo Cloud Native visto en [[KUBERNETES]].

## Errores comunes

- No configurar `spring.task.scheduling.pool.size` y que un `@Scheduled` colgado bloquee a todos los demás (thread único por default).
- Correr `@Scheduled` con múltiples réplicas del Pod sin ningún mecanismo de coordinación — el mismo job se ejecuta N veces en paralelo, uno por instancia.
- Usar `fixedRate` para tareas que pueden tardar más que el intervalo — genera ejecuciones superpuestas si no se controla.
- Elegir Spring Batch para algo trivial, agregando complejidad (Job/Step/Reader/Processor/Writer) donde una query directa alcanzaba.

## Checklist
- [ ] Sé usar `@Scheduled` con `fixedRate`, `fixedDelay` y cron.
- [ ] Entiendo la diferencia práctica entre `fixedRate` y `fixedDelay`.
- [ ] Sé cuándo un caso justifica Spring Batch en vez de un método simple programado.
- [ ] Conozco el problema de ejecutar `@Scheduled` con múltiples réplicas y al menos una forma de resolverlo (Quartz con lock, o CronJob de Kubernetes).
- [ ] Sé para qué sirve chunk processing y por qué importa en volúmenes grandes.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

