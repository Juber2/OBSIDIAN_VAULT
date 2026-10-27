---
tags:
  - Java
  - Cloud
  - Serverless
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - Quarkus
Palabras clave:
  - Lambda, Azure Functions, arranque en frio, SnapStart, CRaC, GraalVM, imagen nativa
---

## 🧠 Concepto

Serverless es ejecutar código **sin gestionar servidores**: la plataforma arranca una instancia cuando llega una petición, la mantiene mientras haya tráfico y la apaga cuando no.

El modelo es excelente para cargas esporádicas. Y Java tiene con él un problema específico: **la JVM tarda en arrancar**.

## ❄️ El arranque en frío

```
Petición → no hay instancia disponible
  1 · La plataforma provisiona el entorno      ~100-200 ms
  2 · Arranca la JVM                            ~200-400 ms
  3 · Carga clases e inicializa Spring          ~2.000-8.000 ms  ← el problema
  4 · Ejecuta el handler                        ~10 ms
```

| Runtime | Arranque en frío típico |
|---|---|
| Node.js / Python | 100-300 ms |
| Go / Rust | 50-150 ms |
| **Java + Spring Boot** | **3.000-8.000 ms** ❌ |
| Java + Micronaut/Quarkus | 800-2.000 ms |
| Java con SnapStart | 200-600 ms |
| **Imagen nativa (GraalVM)** | **20-80 ms** ✅ |

> [!danger] Spring Boot tradicional en Lambda es una mala idea
> Un usuario esperando 6 segundos por la primera petición es inaceptable, y con tráfico esporádico **cada** petición puede ser un arranque en frío. No es un problema de afinación: es la naturaleza de la JVM más un framework que hace mucho trabajo al inicializar.

## 🚀 Las soluciones

### 1 · AWS SnapStart

Toma una **instantánea de la memoria** después de inicializar y la restaura en cada arranque, saltándose toda la fase de inicialización.

```yaml
# SAM
Resources:
  Funcion:
    Type: AWS::Serverless::Function
    Properties:
      Runtime: java21
      SnapStart:
        ApplyOn: PublishedVersions      # ⚠️ solo en versiones publicadas
```

Está basado en **CRaC** (Coordinated Restore at Checkpoint), un proyecto de OpenJDK.

> [!danger] Lo que se restaura desde la instantánea es estado congelado
> Conexiones de red, generadores aleatorios sembrados, timestamps calculados al inicializar: todo eso queda **idéntico en cada restauración**. Un `SecureRandom` sembrado antes de la instantánea produce la misma secuencia en todas las instancias — que es un problema de seguridad serio.

```java
public class MiFuncion implements RequestHandler<Evento, Respuesta>, Resource {

    private Connection conexion;

    public MiFuncion() {
        Core.getGlobalContext().register(this);
    }

    @Override
    public void beforeCheckpoint(Context<? extends Resource> ctx) {
        cerrarRecursos();          // cerrar conexiones ANTES de la instantánea
    }

    @Override
    public void afterRestore(Context<? extends Resource> ctx) {
        reabrirRecursos();         // reabrir DESPUÉS de restaurar
        resembrarAleatoriedad();   // ← imprescindible
    }
}
```

### 2 · Imagen nativa con GraalVM

Compila a un binario nativo: sin JVM, sin carga de clases, sin JIT.

```xml
<!-- Spring Boot 3+ -->
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
</plugin>
```

```bash
./mvnw -Pnative native:compile
```

| Ventaja | Desventaja |
|---|---|
| Arranque en decenas de milisegundos | compilación lenta (5-15 min) |
| Muchísima menos memoria | la reflexión necesita configuración explícita |
| Imagen más chica | menor rendimiento pico (sin JIT) |
| Sin calentamiento | algunas bibliotecas no son compatibles |

> [!warning] La reflexión es el punto de fricción
> La compilación anticipada tiene que saber en tiempo de compilación qué clases se usan por reflexión. Spring Boot 3 genera buena parte de esa configuración automáticamente (AOT), pero las bibliotecas que hacen reflexión dinámica pueden requerir pistas manuales.
> Los errores aparecen **en ejecución**, no al compilar: `ClassNotFoundException` para algo que existe en el código fuente.

```java
@RegisterReflectionForBinding({MiDto.class, OtroDto.class})
@Configuration
public class ConfigNativa { }
```

### 3 · Frameworks pensados para esto

| Framework | Enfoque |
|---|---|
| **Quarkus** | inyección de dependencias resuelta en compilación, nativo de primera clase |
| **Micronaut** | sin reflexión, procesamiento de anotaciones en compilación |
| **Spring Boot 3 + AOT** | procesamiento anticipado, mejora mucho respecto de versiones previas |

> [!tip] Quarkus fue diseñado para este escenario
> Mueve a tiempo de compilación el trabajo que Spring hace al arrancar. Con imagen nativa da arranques de decenas de milisegundos y consumos de memoria varias veces menores — ver [[Quarkus]] y [[08 - Native Image con GraalVM]].

### 4 · Mantener instancias tibias

```yaml
# AWS: concurrencia provisionada
ProvisionedConcurrencyConfig:
  ProvisionedConcurrentExecutions: 5

# Azure Functions: plan Premium con instancias precalentadas
```

> [!warning] Esto elimina el ahorro del modelo
> Pagás por instancias encendidas esperando tráfico, que es exactamente lo que serverless prometía evitar. Si terminás con concurrencia provisionada permanente, **probablemente querías un contenedor** — ver [[02 - Computo para Apps Java]].

## 📐 Los límites

| Límite | AWS Lambda | Azure Functions |
|---|---|---|
| Duración máxima | 15 min | 5 min (consumo) · sin límite (Premium) |
| Memoria | hasta 10 GB | hasta 14 GB (Premium) |
| Tamaño del paquete | 50 MB comprimido · 250 MB descomprimido | similar |
| Almacenamiento temporal | `/tmp`, hasta 10 GB | `/tmp` acotado |
| Concurrencia | límite por cuenta, ampliable | según plan |

> [!danger] El estado en memoria sobrevive entre invocaciones — a veces
> La plataforma reutiliza la instancia si hay tráfico. Una variable estática puede conservar valor entre invocaciones... o no. **Nunca dependas de eso** para corrección; sí podés usarlo como caché oportunista.

```java
public class MiFuncion implements RequestHandler<Evento, Respuesta> {

    // Se inicializa una vez por instancia: aprovecha el reúso
    private static final ClienteBase cliente = ClienteBase.crear();

    // ❌ NUNCA para estado de negocio
    private static final Map<String, Datos> cache = new HashMap<>();
}
```

> [!warning] El pool de conexiones y serverless se llevan mal
> Cada instancia abre sus propias conexiones. Con 500 invocaciones concurrentes, son 500 conexiones a una base que admite 100. Las salidas: un agrupador externo (RDS Proxy), o una base con modelo de acceso HTTP (DynamoDB, Cosmos DB).

## 🎯 Cuándo conviene

| Caso | ¿Serverless? |
|---|---|
| Tarea programada esporádica | ✅ ideal |
| Procesar archivos al subirse | ✅ ideal |
| Webhook de baja frecuencia | ✅ |
| Transformar eventos de una cola | ✅ |
| API con tráfico irregular | ⚠️ ojo con el arranque en frío |
| API con tráfico constante | ❌ un contenedor sale más barato |
| Proceso largo (> 15 min) | ❌ |
| Servicio con estado o pool grande | ❌ |

> [!tip] La regla del cruce de costos
> Serverless cobra por invocación y por tiempo de ejecución; un contenedor cobra por estar encendido. **Con tráfico constante, el contenedor gana casi siempre.** Con tráfico esporádico, serverless puede costar centavos donde un contenedor cuesta decenas de dólares.
> Hacé la cuenta con tu volumen real antes de decidir.

## 💻 Escribir la función

```java
// AWS Lambda
public class ProcesarDocumento implements RequestHandler<S3Event, String> {

    private static final ProcesadorDocumentos procesador = new ProcesadorDocumentos();

    @Override
    public String handleRequest(S3Event evento, Context contexto) {
        for (var registro : evento.getRecords()) {
            String clave = registro.getS3().getObject().getUrlDecodedKey();
            procesador.procesar(clave);           // idempotente: puede reentregarse
        }
        return "OK";
    }
}
```

```java
// Azure Functions
public class ProcesarDocumento {

    @FunctionName("procesarDocumento")
    public void ejecutar(
            @BlobTrigger(name = "archivo", path = "documentos/{nombre}",
                         connection = "AzureWebJobsStorage") byte[] contenido,
            @BindingName("nombre") String nombre,
            final ExecutionContext contexto) {
        procesador.procesar(nombre, contenido);
    }
}
```

> [!danger] Las funciones deben ser idempotentes
> Los disparadores de eventos son **at-least-once**: la misma función puede invocarse dos veces con el mismo evento. Si procesa un archivo dos veces, duplica registros. Aplica todo [[03 - Idempotencia y Deduplicacion]].

## 💰 El costo

```
Lambda: (invocaciones × precio) + (GB-segundo × precio)

1 M de invocaciones/mes, 512 MB, 200 ms cada una
= 1M × precio_invocación + (1M × 0,5 GB × 0,2 s) × precio_GB_s
```

> [!tip] Más memoria puede salir más barato
> La CPU asignada es proporcional a la memoria. Duplicar la memoria puede reducir el tiempo de ejecución a la mitad, y el costo total queda igual — con la mitad de latencia. Para Java, asignar poca memoria suele ser contraproducente.
> Vale medir con distintos tamaños en vez de asumir que menos memoria es más barato.

## 🧪 Tests

```java
@Test
void la_funcion_procesa_el_evento() {
    var respuesta = funcion.handleRequest(eventoDePrueba(), contextoSimulado());
    assertEquals("OK", respuesta);
}

@Test
void procesar_el_mismo_evento_dos_veces_no_duplica() {
    funcion.handleRequest(evento, contexto);
    funcion.handleRequest(evento, contexto);
    assertEquals(1, repo.count());
}

@Test
void el_arranque_en_frio_esta_dentro_del_presupuesto() {
    // medido contra el binario nativo o con SnapStart
    assertTrue(medirArranqueEnFrio().toMillis() < 500);
}

@Test
void tras_restaurar_la_instantanea_la_aleatoriedad_es_distinta() {
    // el riesgo específico de SnapStart/CRaC
}
```

Con **LocalStack** se prueba Lambda localmente; Azure Functions tiene su propio runtime local (`func start`).

## ⚠️ Errores Comunes

> [!warning] Los que hacen fracasar el enfoque
> - **Spring Boot tradicional en Lambda**: arranques en frío de segundos.
> - **Pool de conexiones grande**: agota el límite de la base.
> - **Estado en memoria** asumiendo que la instancia se reutiliza.
> - **`SecureRandom` sembrado antes de la instantánea** en SnapStart.
> - **Conexiones abiertas al tomar la instantánea**: quedan inservibles al restaurar.
> - **Funciones no idempotentes** con disparadores at-least-once.
> - **Poca memoria para ahorrar**: sale más caro por el tiempo extra.
> - **Concurrencia provisionada permanente**: entonces querías un contenedor.
> - **Procesos largos** contra el límite de duración.

## 📌 Notas Rápidas
- La decisión honesta: **si el tráfico es constante, no uses serverless para el servicio principal**. Usalo para lo que realmente es esporádico o dirigido por eventos, que es donde brilla.
- El combo que hace viable Java en serverless es **Quarkus o Micronaut compilado a imagen nativa**: arranques de decenas de milisegundos y consumo de memoria muy bajo. Spring Boot 3 con AOT mejoró bastante pero sigue por detrás.
- El riesgo de la aleatoriedad con SnapStart es de esos detalles que casi nadie conoce y que en banca importa muchísimo: mismas semillas en todas las instancias significa tokens predecibles.
- Se relaciona con [[08 - Native Image con GraalVM]] y [[02 - Quarkus vs Spring Boot]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[10 - Red y Exposicion]]
- **Siguiente**: [[12 - Infraestructura como Codigo]]
- **Se conecta con**: [[Quarkus]], [[02 - Computo para Apps Java]], [[03 - App Service y Azure Functions]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

