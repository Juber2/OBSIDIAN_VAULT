---
tags:
  - Java
  - Cloud
  - Azure
  - AWS
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
  - 01 - Modelo Mental Cloud
Palabras clave:
  - App Service, Container Apps, ECS, Fargate, AKS, EKS, escalado, JVM, cgroups
---

## 🧠 Concepto

La pregunta es simple: **¿dónde corre tu JAR?** Las respuestas van de "subí el archivo y listo" a "operá un clúster de Kubernetes", y la diferencia de esfuerzo operativo entre los extremos es enorme.

> [!tip] Elegí el nivel más alto que resuelva tu problema
> Cada escalón hacia abajo te da más control y te transfiere más trabajo. Kubernetes es la respuesta correcta para muchas organizaciones y la respuesta cara para la mayoría de las aplicaciones.

## 🪜 Las opciones

| Nivel | Qué gestionás | Azure | AWS |
|---|---|---|---|
| **PaaS** | solo el código | App Service | Elastic Beanstalk |
| **Contenedores gestionados** | la imagen | Container Apps | ECS Fargate · App Runner |
| **Kubernetes** | el clúster | AKS | EKS |
| **Serverless** | la función | Azure Functions | Lambda |
| **IaaS** | todo | Máquinas virtuales | EC2 |

### PaaS — subí el JAR

```bash
# Azure App Service
az webapp deploy --resource-group rg-corebank --name corebank --src-path target/app.jar --type jar

# AWS Elastic Beanstalk
eb deploy
```

| Ventaja | Desventaja |
|---|---|
| Cero infraestructura que operar | menos control del entorno |
| Despliegue en un comando | escalado menos afinable |
| Certificados y dominios integrados | difícil de replicar en local |
| Ranuras de despliegue integradas | ata al proveedor |

> [!tip] Para una aplicación sola, PaaS suele ser la respuesta correcta
> Si tenés un servicio, sin necesidades raras de red, PaaS te da el 90% del valor con el 10% del trabajo. El salto a Kubernetes se justifica cuando hay muchos servicios que coordinar.

### Contenedores gestionados — el punto dulce

```bash
# Azure Container Apps
az containerapp create --name corebank --image acr.azurecr.io/corebank:1.4.2 \
  --min-replicas 2 --max-replicas 20 --target-port 8080

# AWS ECS Fargate: se define una tarea y un servicio; sin servidores que administrar
aws ecs update-service --cluster corebank --service api --desired-count 4
```

| Ventaja | Desventaja |
|---|---|
| La imagen corre igual en cualquier lado | hay que construir imágenes |
| Sin nodos que parchear | menos control que Kubernetes |
| Escalado automático incluido | funciones más limitadas |

> [!tip] Es donde termina la mayoría de los proyectos
> Contenedores (portables, reproducibles) sin la carga operativa de un clúster. Azure Container Apps y ECS Fargate cubren casi todo lo que un backend necesita.

### Kubernetes — cuando se justifica

| Se justifica si… | No se justifica si… |
|---|---|
| muchos servicios con orquestación compleja | tenés uno o dos servicios |
| necesitás portabilidad entre nubes | no vas a cambiar de nube |
| hay equipo con experiencia real | nadie lo operó antes |
| requisitos de red o afinidad específicos | el caso es estándar |

> [!danger] Kubernetes tiene un costo operativo permanente
> Actualizaciones del clúster, versiones de las API que se deprecan, redes, políticas, RBAC, observabilidad propia. Adoptarlo "porque es el estándar" con un equipo chico es una de las decisiones que más tiempo consume sin retorno.

## ⚙️ La JVM en un contenedor

El problema específico de Java en la nube, y el que más incidentes causa.

```dockerfile
# ❌ Memoria fija: cambiar el límite del contenedor obliga a reconstruir
ENV JAVA_OPTS="-Xmx512m"

# ✅ Porcentaje: la JVM se adapta al límite del cgroup
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75 -XX:InitialRAMPercentage=50"
```

> [!danger] La JVM lee el límite del contenedor, no el de la máquina
> Desde Java 10 el soporte de cgroups está activo por defecto (`UseContainerSupport`), así que la JVM ve el límite del contenedor. Pero si fijás `-Xmx` a mano y el límite del contenedor es menor, **el orquestador mata el proceso por exceso de memoria** antes de que el recolector de basura llegue a actuar.
> El síntoma es el peor posible: `OOMKilled` sin excepción, sin log y sin volcado. El contenedor simplemente desaparece.

### Por qué 75% y no 100%

La JVM usa memoria fuera del heap que también cuenta para el límite del contenedor:

```
Límite del contenedor: 1 GB
├── Heap (MaxRAMPercentage 75%)     ~750 MB
├── Metaspace                        ~80 MB
├── Pilas de hilos                   ~50 MB
├── Buffers directos y código nativo ~80 MB
└── margen                           ~40 MB
```

> [!warning] Dejá siempre margen fuera del heap
> Poner `MaxRAMPercentage=100` garantiza `OOMKilled`. Entre 65% y 75% es el rango razonable; con muchos hilos o mucha memoria nativa, más bajo.

```dockerfile
ENV JAVA_OPTS="\
  -XX:MaxRAMPercentage=75 \
  -XX:+UseG1GC \
  -XX:+ExitOnOutOfMemoryError \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/dumps"
```

> [!tip] `ExitOnOutOfMemoryError` es imprescindible en la nube
> Sin él, una JVM sin memoria queda **viva pero inservible**: responde el health check y no procesa nada. El orquestador no la reemplaza porque cree que está sana. Con el flag, el proceso muere y es reemplazado de inmediato.

### CPU

```
Límite de CPU: 1 núcleo
→ Runtime.availableProcessors() = 1
→ el pool de ForkJoin, el de G1 y el de Tomcat se dimensionan para 1
```

> [!warning] Limitar la CPU a menos de 1 núcleo perjudica mucho a la JVM
> Con `cpu: 0.5` el proceso sufre *throttling* del cgroup en plena pausa de recolección de basura, y las latencias se disparan de forma errática. Para aplicaciones Java conviene **al menos 1 núcleo completo**, incluso si el promedio de uso es bajo.

## 📈 Escalado automático

```yaml
# Azure Container Apps: escala por peticiones concurrentes
--scale-rule-name http-rule
--scale-rule-type http
--scale-rule-http-concurrency 50
--min-replicas 2
--max-replicas 20
```

```json
// AWS ECS: escala por métrica
{
  "TargetTrackingScalingPolicyConfiguration": {
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": { "PredefinedMetricType": "ECSServiceAverageCPUUtilization" },
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60
  }
}
```

| Métrica | Buena para |
|---|---|
| CPU | procesamiento intensivo |
| Peticiones concurrentes | APIs HTTP ✅ |
| Longitud de la cola | consumidores de eventos ✅ |
| Latencia | cuando el SLA es de latencia |

> [!tip] Para una API, escalá por concurrencia, no por CPU
> Una aplicación Java típica está esperando E/S la mayor parte del tiempo: la CPU se mantiene baja mientras las peticiones se acumulan. Escalar por CPU reacciona tarde o directamente nunca.

> [!danger] `min-replicas: 0` y Java no se llevan bien
> Escalar a cero ahorra dinero y hace que la primera petición después de un período inactivo espere el arranque completo de la JVM — segundos, a veces decenas. Para una API con usuarios, el mínimo es 2 (uno no tolera el reemplazo durante un despliegue).

### Arranque rápido

```yaml
spring:
  main:
    lazy-initialization: true      # ⚠️ los errores de configuración aparecen tarde
  jmx:
    enabled: false
```

```dockerfile
# Class Data Sharing: acelera el arranque precompilando metadatos de clases
RUN java -XX:ArchiveClassesAtExit=/app/app.jsa -jar app.jar --spring.context.exit=onRefresh
ENV JAVA_OPTS="$JAVA_OPTS -XX:SharedArchiveFile=/app/app.jsa"
```

Para arranques realmente rápidos, la respuesta es imagen nativa — ver [[11 - Serverless con Java]].

## 🩺 Sondas de salud

```yaml
management:
  endpoint:
    health:
      probes:
        enabled: true
  health:
    livenessState:  { enabled: true }
    readinessState: { enabled: true }
```

| Sonda | Endpoint | Si falla |
|---|---|---|
| **Liveness** | `/actuator/health/liveness` | se reinicia el contenedor |
| **Readiness** | `/actuator/health/readiness` | se saca del balanceador |
| **Startup** | `/actuator/health/liveness` con más tolerancia | espera al arranque |

> [!danger] La base de datos nunca va en el liveness
> Si Postgres se cae, el liveness falla en todas las instancias, el orquestador las reinicia todas, y al volver siguen sin base. Reiniciar en bucle no arregla una dependencia caída — y además borra el estado que permitiría diagnosticarla.
> **La base va en el readiness**: la instancia deja de recibir tráfico pero sigue viva.

```java
@Component
public class BaseDatosReadiness implements HealthIndicator {
    @Override public Health health() {
        return conexionOk() ? Health.up().build() : Health.down().build();
    }
}
```

## 🎯 Cómo decidir

```
¿Es una función corta, esporádica y sin estado?
├── SÍ → serverless (ojo con el arranque en frío de la JVM)
└── NO → ¿tenés muchos servicios que coordinar?
         ├── SÍ → ¿hay equipo con experiencia en Kubernetes?
         │        ├── SÍ → AKS / EKS
         │        └── NO → contenedores gestionados
         └── NO → ¿necesitás contenedores?
                  ├── SÍ → Container Apps / ECS Fargate  ← el punto dulce
                  └── NO → App Service / Beanstalk
```

## ⚠️ Errores Comunes

> [!warning] Los que producen incidentes en producción
> - **`-Xmx` fijo** en vez de `MaxRAMPercentage`: `OOMKilled` silencioso.
> - **`MaxRAMPercentage=100`**: no deja margen para lo que está fuera del heap.
> - **Sin `ExitOnOutOfMemoryError`**: la JVM queda viva e inservible.
> - **Menos de 1 núcleo de CPU**: *throttling* durante las pausas de GC.
> - **Base de datos en el liveness**: bucle de reinicios.
> - **Escalar por CPU** una API que espera E/S.
> - **`min-replicas: 0`** con JVM: primera petición lentísima.
> - **Una sola réplica**: cada despliegue es una caída.
> - **Kubernetes sin equipo que lo opere.**

## 📌 Notas Rápidas
- La combinación mínima para Java en contenedores: `MaxRAMPercentage=75`, `ExitOnOutOfMemoryError`, `HeapDumpOnOutOfMemoryError`, al menos 1 núcleo, liveness y readiness separados. Con eso se evitan la mayoría de los incidentes típicos.
- Azure tuvo un servicio dedicado a Spring (Azure Spring Apps) que está en camino de retiro; para nuevos proyectos conviene ir directo a Container Apps o AKS. Verificá el estado actual antes de elegirlo.
- `lazy-initialization: true` acelera el arranque a cambio de que los errores de configuración aparezcan en la primera petición y no al arrancar. En producción suele no compensar.
- Se aplica en [[Fase 7 - Observabilidad y Despliegue]].

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]]
- **Anterior**: [[01 - Modelo Mental Cloud]]
- **Siguiente**: [[03 - Contenedores y Registries]]
- **Se conecta con**: [[01 - Docker con Java]], [[04 - JVM Memoria y GC]], [[KUBERNETES]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

