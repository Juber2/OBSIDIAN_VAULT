---
tags:
  - Postulacion
  - ISPConsulting
  - Java
  - SpringBoot
  - Angular
  - GCP
  - Kubernetes
  - Mensajeria
Creado: 2026-08-30
Estado: Por evaluar
Empresa: ISP Consulting
Modalidad: Híbrido
Ubicacion: San Borja, Lima
Sueldo: S/5,000-S/5,500
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - JAVA
  - ANGULAR
  - KUBERNETES
---

# ISP Consulting — Java, Angular, GCP y mensajería

## Veredicto rápido

**Postular: sí, si deseas mantener Java como segunda vía profesional; prioridad media y potencialmente alta según tu experiencia comprobable.**

Aunque el título diga Full Stack, el perfil real es principalmente un **backend/cloud senior con Java y Spring**, especializado en microservicios, mensajería asíncrona y Kubernetes. Angular es importante, pero parece complementario al núcleo backend.

La vacante es técnicamente más coherente que la anterior: sus herramientas pertenecen a una misma arquitectura distribuida. Sin embargo, solicita alrededor de cuatro años en Java, Spring Boot, microservicios, JUnit, Swagger y Angular; tres años en cloud/Kubernetes y mensajería. El sueldo de **S/5,000-S/5,500** resulta ajustado para esa combinación de experiencia y responsabilidad técnica.

## Encaje estimado por escenario

| Experiencia comprobable | Encaje aproximado |
|---|---:|
| 4 años Java/Spring, Angular y microservicios; Kubernetes/mensajería reales | 80-90 % |
| Java/Spring sólido y Angular profesional; brechas en GCP o algún broker | 68-80 % |
| Java y Angular sólidos en proyectos, pero poca operación cloud real | 55-70 % |
| Perfil principal .NET/Angular con Java mayormente académico | 40-58 % |
| Menos de cuatro años Java/Spring y la empresa usa el tiempo como filtro rígido | Riesgo alto |

El anuncio dice “requisitos obligatorios”, pero en la sección de experiencia marca los años como “deseables”. Conviene postular si puedes defender el núcleo y aclarar con reclutamiento si los años son filtro excluyente.

## Perfil real de la vacante

```text
Angular + TypeScript
        │
        ▼
APIs Java 11 + Spring Boot
        │
        ├── SQL / NoSQL
        ├── Pub/Sub, RabbitMQ o Kafka
        ├── observabilidad
        └── contenedores en Kubernetes
                     │
                     └── GCP y/o AWS
```

La expectativa de “liderar decisiones técnicas” no necesariamente implica gestión de personas. Parece liderazgo de diseño, rendimiento y prácticas, por lo que se deben preparar decisiones y trade-offs más que historias de evaluación de personal.

## Matriz de encaje

| Requisito | Importancia | Evidencia necesaria |
|---|---|---|
| Java 11 | Crítica | código profesional, concurrencia, JVM, errores y performance |
| Spring Boot | Crítica | API completa, datos, seguridad, pruebas y Actuator |
| Microservicios | Crítica | límites, contratos, fallos, datos y operación |
| Mensajería asíncrona | Crítica | entrega, retries, DLQ, idempotencia y orden |
| Kubernetes | Crítica | deploy, services, config, secrets, probes y diagnóstico |
| GCP/AWS | Alta pero ambigua | despliegue y operación, no solo conceptos |
| SQL/NoSQL | Alta | modelado, índices, transacciones y elección justificada |
| Angular/TypeScript | Alta | feature moderna integrada con backend |
| JUnit | Alta | unit tests útiles e integration tests |
| Observabilidad/performance | Alta | métricas, trazas, logs y diagnóstico |
| Swagger/Postman | Media | contrato, collections y pruebas de API |
| HTML/CSS/JavaScript | Base | implementación responsive y fundamentos web |

## Java 11 y Spring — pregunta crítica de versión

Java 11 específico suele indicar una plataforma existente. Debes repasar:

- colecciones, generics, Streams, Optional y excepciones;
- lambdas, interfaces funcionales y `CompletableFuture`;
- memoria, garbage collection, thread pools y diagnóstico;
- `HttpClient`, APIs de archivos y utilidades incorporadas hasta Java 11;
- Maven o Gradle, dependencias, profiles y configuración;
- migración desde Java 8 y dependencias que usan APIs removidas.

La guía oficial de Oracle cubre los cambios y riesgos al migrar aplicaciones hacia JDK 11: [Oracle JDK 11 Migration Guide](https://docs.oracle.com/en/java/javase/11/migrate/).

Debes preguntar la versión exacta de Spring Boot. Spring Boot 3 y 4 requieren como mínimo Java 17, mientras que Spring Boot 2.7 es compatible con Java 11. Por eso, “Java 11 + Spring” probablemente describe una línea 2.x o una restricción legacy, pero debe confirmarse: [requisitos actuales de Spring Boot](https://docs.spring.io/spring-boot/system-requirements.html) y [requisitos de Spring Boot 2.7](https://docs.spring.io/spring-boot/docs/2.7.x/reference/html/getting-started.html).

No prepares respuestas usando exclusivamente características de Java 17/21 si la prueba se ejecutará en Java 11.

Material: [[JAVA]], [[Spring Boot]], [[Java Intermedio]], [[Java Avanzado]], [[Performance Java]].

## Spring Boot que debes defender

- IoC/DI, scopes y configuración.
- Controllers REST, DTOs, validación y errores.
- Spring Data/JPA, transacciones y migrations.
- Spring Security si las APIs están protegidas.
- clientes HTTP con timeout y resiliencia.
- Actuator, health checks, métricas y trazas.
- JUnit 5, Mockito, integration tests y Testcontainers.
- empaquetado, configuración por ambiente y graceful shutdown.

Material: [[01 - Fundamentos Spring Boot]], [[02 - REST Controllers]], [[04 - Validaciones]], [[08 - Manejo de Errores]], [[Spring Data]], [[Spring Security]], [[Testing en Java]].

## Mensajería asíncrona — brecha prioritaria

La vacante enumera Pub/Sub, RabbitMQ y Kafka. Antes de memorizar APIs, domina los problemas comunes:

- producer, consumer, topic/queue, subscription y acknowledgment;
- entrega at-most-once, at-least-once y las condiciones reales de exactly-once;
- idempotencia del consumidor;
- retries con backoff y límite;
- dead-letter queue y redrive;
- ordenamiento y particionamiento;
- duplicados, poison messages y schema evolution;
- consumer groups, concurrencia y backpressure;
- observabilidad, lag, throughput y alertas;
- Outbox para coordinar base de datos y publicación;
- seguridad, permisos y datos sensibles.

### Diferencias que debes explicar

| Tecnología | Modelo dominante | En qué enfocarte |
|---|---|---|
| Google Pub/Sub | servicio administrado publisher/subscriber | topics, subscriptions, ack, push/pull, IAM, DLQ |
| RabbitMQ | broker de colas y routing | exchanges, queues, bindings, ack, prefetch y DLX |
| Kafka | log distribuido particionado | topics, partitions, offsets, consumer groups y retención |

Google describe Pub/Sub como mensajería asíncrona que desacopla productores y consumidores; topics y subscriptions permiten distribuir cada mensaje y requieren acknowledgment del consumidor: [documentación oficial de Pub/Sub](https://docs.cloud.google.com/pubsub/docs/pubsub-basics).

No afirmar que las tres tecnologías son intercambiables. Explica qué necesidad favorece cada una y confirma cuáles usa realmente el proyecto.

Material: [[Mensajeria]], [[01 - Kafka Fundamentos]], [[02 - Kafka con Spring Boot]], [[03 - RabbitMQ Fundamentos]], [[04 - RabbitMQ con Spring Boot]], [[07 - Comunicacion Asincrona - Kafka]].

## Microservicios con responsabilidad operativa

Debes poder responder:

- cómo definir límites por capacidad de negocio;
- cuándo preferir un monolito modular;
- cómo versionar contratos y evitar cambios incompatibles;
- cómo manejar consistencia sin transacciones distribuidas generales;
- cómo decidir entre REST y eventos;
- dónde aplicar idempotencia;
- cómo propagar identidad y trace context;
- cómo probar servicios y contratos;
- cómo desplegar, observar y revertir un servicio.

Material: [[Microservicios]], [[01 - Fundamentos Microservicios]], [[05 - Circuit Breaker - Resilience4j]], [[08 - Distributed Tracing]], [[05 - Proyecto Microservicios]].

## Kubernetes, GCP y AWS

Kubernetes aparece por primera vez de forma explícita, pero es crítico para esta vacante. El laboratorio mínimo debe incluir:

1. imagen multi-stage de un servicio Spring Boot;
2. Deployment y Service;
3. ConfigMap y Secret;
4. readiness, liveness y startup probes;
5. requests/limits;
6. rolling update y rollback;
7. logs y métricas;
8. diagnóstico de pod reiniciándose o sin conectividad;
9. escalado y comportamiento ante carga;
10. una dependencia externa, sin ejecutar la base de producción dentro del clúster por defecto.

GKE es la implementación administrada de Kubernetes en Google Cloud para desplegar y operar aplicaciones contenerizadas; la plataforma administra partes como el control plane y, según el modo, también los nodos: [descripción oficial de GKE](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview).

Material: [[KUBERNETES]], [[01 - Fundamentos y Arquitectura]], [[02 - Pods Deployments y ReplicaSets]], [[03 - Services y Networking]], [[04 - ConfigMaps Secrets y Volumenes]], [[05 - Ingress y Exposicion Externa]], [[06 - Namespaces RBAC y Seguridad Basica]], [[04 - Kubernetes para Spring Boot]].

### Qué aclarar de cloud

La redacción “Kubernetes con GCP-AWS” es ambigua:

- puede exigir GKE y EKS;
- puede significar experiencia en cualquiera de los dos;
- o puede indicar migración/multicloud.

No necesitas construir dos plataformas iguales antes de hablar con la empresa. Aprende Kubernetes de forma transferible y despliega primero en el proveedor principal confirmado.

## SQL y NoSQL

“SQL y NoSQL” no identifica productos. Preparar:

- modelado relacional, joins, índices, planes y transacciones;
- aislamiento, locks y concurrencia;
- connection pools y consultas lentas desde Java;
- migrations con Flyway/Liquibase;
- modelado documental o clave-valor según acceso;
- consistencia, particionado y límites del producto;
- justificar por qué una entidad pertenece a SQL o NoSQL.

No usar NoSQL solo porque la arquitectura tiene microservicios.

Material: [[Bases de Datos Java]], [[01 - SQL para Backend Java]], [[03 - Indices y Queries Lentas]], [[05 - NoSQL desde Java]].

## Angular y fundamentos frontend

Aunque el anuncio repite cuatro años para Angular, el texto principal le dedica menos profundidad que al backend. Debes demostrar:

- TypeScript estricto y JavaScript moderno;
- componentes standalone y arquitectura por features;
- routing, forms, HttpClient e interceptors;
- RxJS y Signals;
- auth, manejo de errores y estados de UI;
- HTML semántico, CSS responsive y accesibilidad;
- unit tests y una feature integrada con Java.

Material: [[ANGULAR]], [[JavaScript moderno para Angular]], [[Feature Architecture Boundaries]], [[Testing Strategy Angular]].

## Observabilidad y rendimiento

Preparar un flujo de diagnóstico completo:

```text
síntoma del usuario
  -> métricas de latencia/error/tráfico/saturación
  -> trace Angular/API/mensajería/DB
  -> logs estructurados con correlation ID
  -> perfil de JVM, threads, heap o GC
  -> consulta lenta, consumer lag o throttling cloud
  -> mitigación, causa raíz y prevención
```

No reducir observabilidad a imprimir logs. Debes conectar métricas, trazas y logs con SLO, alertas útiles e incident response.

Material: [[06 - Logs Observabilidad y Monitoreo]], [[04 - Performance en Spring Boot]], [[01 - Profiling y Diagnostico]], [[05 - Incidentes y Debugging]].

## JUnit, Swagger y Postman

### JUnit

- unit tests rápidos para reglas;
- Mockito solo en límites útiles;
- integration tests para HTTP, persistencia y mensajería;
- Testcontainers para dependencias reales;
- contract tests y smoke tests;
- evitar tests que solamente repiten la implementación.

### Swagger/OpenAPI

- contrato antes o junto con implementación;
- schemas, errores, auth y ejemplos;
- versionado y compatibilidad;
- generación o validación automática en CI.

### Postman

- environments sin secretos en el repositorio;
- collection organizada;
- scripts para auth y aserciones;
- casos positivos y negativos;
- ejecución por CLI en pipeline si el equipo lo usa.

Postman es una herramienta, no una competencia de arquitectura. Un año de uso se cubre rápido si ya comprendes HTTP y APIs.

Material: [[01 - JUnit 5]], [[04 - Integration Testing]], [[05 - TestContainers]], [[09 - Swagger y OpenAPI]].

## Proyecto demostrable alineado

Crear un sistema pequeño de pedidos o notificaciones:

- Angular para registrar y consultar pedidos.
- API Java 11 + Spring Boot compatible.
- PostgreSQL como almacenamiento principal.
- evento `OrderCreated` mediante un puerto de mensajería.
- implementación local con RabbitMQ o Kafka.
- una segunda implementación o laboratorio con Pub/Sub.
- consumidor idempotente, retry y DLQ.
- OpenAPI y colección Postman.
- JUnit, integration tests y Testcontainers.
- contenedores desplegados en Kubernetes.
- logs, métricas, traces y dashboard mínimo.
- README con decisiones y comparación de brokers.

No necesitas desplegar Kafka, RabbitMQ y Pub/Sub simultáneamente. Demuestra una implementación profunda y una comparación técnicamente correcta.

## Preparación de siete días

| Día | Foco | Entregable |
|---|---|---|
| 1 | Evidencia y CV | matriz de años y cuatro historias técnicas |
| 2 | Java 11 + Spring | API, datos, validación, errores y Actuator |
| 3 | Mensajería | consumidor idempotente, retry y DLQ |
| 4 | Kubernetes | deployment, probes, config, limits y rollback |
| 5 | GCP | Pub/Sub + conceptos GKE/IAM/observabilidad |
| 6 | Angular | feature integrada, tests y errores |
| 7 | System design | diseño completo, performance e entrevista simulada |

Postular antes de finalizar el plan.

## Preguntas probables

### Java/Spring

- ¿Qué particularidades debes considerar al trabajar con Java 11?
- ¿Cómo diagnosticas alto consumo de CPU o memoria?
- ¿Cómo manejas transacciones y concurrencia?
- ¿Cómo pruebas un controller, un servicio y un repositorio?
- ¿Qué versión de Spring Boot es compatible con el runtime del proyecto?

### Mensajería

- ¿Qué ocurre si el consumidor procesa y falla antes del ack?
- ¿Cómo garantizas idempotencia?
- ¿Cuándo elegir Kafka frente a RabbitMQ o Pub/Sub?
- ¿Cómo manejas mensajes venenosos?
- ¿Cómo preservas orden sin destruir el throughput?
- ¿Cómo publicas un evento junto con una transacción de base de datos?

### Kubernetes/cloud

- Diferencia entre readiness y liveness.
- ¿Cómo investigas un `CrashLoopBackOff`?
- ¿Cómo manejas secretos y configuración?
- ¿Cómo haces rollback?
- ¿Qué observarías en un consumer que se está retrasando?
- ¿Qué diferencias operativas esperas entre GKE y EKS?

### Arquitectura

- ¿Cuándo no usar microservicios?
- ¿Cómo decides el límite de un servicio?
- ¿Cómo mantienes compatibilidad de una API o evento?
- ¿Cómo reduces latencia sin sacrificar consistencia?
- Describe una decisión técnica que lideraste.

## Palabras clave para el CV

`Java 11`, `Spring Boot`, `Spring Data`, `REST APIs`, `microservices`, `event-driven architecture`, `Kafka`, `RabbitMQ`, `Google Pub/Sub`, `Kubernetes`, `GKE`, `AWS`, `SQL`, `NoSQL`, `Angular`, `TypeScript`, `JUnit 5`, `Testcontainers`, `OpenAPI`, `Postman`, `observability`, `performance tuning`.

Solo incluir tecnologías con experiencia o proyecto defendible. Diferenciar “experiencia profesional”, “proyecto” y “conocimiento”.

## Preguntas para reclutamiento

1. ¿Los cuatro años son excluyentes o deseables?
2. ¿Qué porcentaje del puesto es Java/Spring y qué porcentaje Angular?
3. ¿Qué versión de Spring Boot utilizan con Java 11?
4. ¿Existe plan de migración a un JDK más reciente?
5. ¿Usan Pub/Sub, RabbitMQ y Kafka simultáneamente o depende del proyecto?
6. ¿La experiencia cloud debe ser en GCP y AWS, o basta uno de los dos?
7. ¿Utilizan GKE, EKS o Kubernetes autogestionado?
8. ¿Qué base SQL y qué producto NoSQL emplean?
9. ¿“Liderar decisiones técnicas” implica rol de referente, arquitectura o gestión de personas?
10. ¿Existe guardia/on-call o soporte fuera de horario?
11. ¿Cuántos días presenciales requiere San Borja?
12. ¿El rango salarial es negociable según los años exigidos?

## Decisión final

La vacante es una **buena postulación secundaria para consolidar Java + Angular** y una excelente fuente de prioridades técnicas: mensajería, Kubernetes y observabilidad. Puede ser prioridad alta si acreditas los años de Java/Spring y deseas una vía backend cloud.

No desplaza automáticamente la ruta Angular + .NET + Azure, pero Java/Spring y Angular llegan ambos a tres apariciones en el radar. Eso confirma que Java ya no es una excepción: es una segunda ruta laboral real. La mayor cautela es económica y documental, no tecnológica.
