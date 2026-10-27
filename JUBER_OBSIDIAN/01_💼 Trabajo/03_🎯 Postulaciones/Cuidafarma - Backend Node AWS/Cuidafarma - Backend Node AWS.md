---
tags:
  - Postulacion
  - Cuidafarma
  - Backend
  - NodeJS
  - AWS
  - PostgreSQL
Creado: 2026-08-30
Estado: Por postular
Empresa: Cuidafarma - Grupo Breca
Modalidad: Presencial
Ubicación: Lima
Sueldo: S/ 6,500 - S/ 7,500
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - NESTJS
  - POSTGRESQL
---

# Cuidafarma — Desarrollador Backend Node.js + AWS

## Veredicto rápido

**Postular: sí como oportunidad stretch, si aceptas presencialidad y cumples tres años de experiencia backend.**

Es una vacante técnicamente coherente y más cercana a ingeniería backend/cloud que a Full Stack Angular. No solicita Angular, .NET ni Azure. Sus dos filtros más fuertes son:

1. Node.js/JavaScript intermedio-avanzado.
2. Experiencia práctica en AWS serverless.

No conviene presentarte como especialista AWS si solo conoces equivalencias desde Azure. Sí puedes postular destacando experiencia backend transferible, Node/NestJS si es real, PostgreSQL, APIs, eventos, seguridad, testing y operación.

## Encaje estimado por escenario

| Situación real | Encaje aproximado |
|---|---:|
| Tienes 3+ años backend y experiencia práctica con Node/NestJS, pero poca AWS | 55-65 % |
| Tienes 3+ años backend principalmente .NET/Java y solo práctica académica Node | 40-50 % |
| No llegas a tres años backend comprobables | Menor por filtro de experiencia |
| Ya usaste Lambda/SQS/RDS/CloudWatch en un proyecto real | 70 % o más |

El vault demuestra que existe material relevante, pero una nota no prueba experiencia profesional. Completar la matriz con evidencia real antes de adaptar el CV.

## Matriz de encaje

| Requisito | Encaje documental | Brecha real a cerrar |
|---|---|---|
| Node.js y JavaScript avanzado | 🟡 Parcial | Dominar runtime sin depender solo de NestJS |
| APIs REST y OpenAPI | 🟢 Alto | Preparar ejemplos y contratos propios |
| Lambda | 🔴 Bajo/no evidenciado | Crear funciones y entender invocaciones |
| API Gateway | 🟡 Conceptual | Implementar rutas, auth, throttling y errores |
| S3 | 🟡 Transferible desde Blob | Practicar SDK, presigned URLs y eventos |
| SQS y DLQ | 🟡 Conceptual | Implementar consumidor idempotente |
| EventBridge | 🔴 Bajo/no evidenciado | Aprender bus, reglas, targets y retries |
| RDS | 🟡 Transferible | Practicar PostgreSQL y conexiones desde Lambda |
| CloudWatch | 🟡 Transferible | Logs Insights, métricas, alarmas y trazabilidad |
| PostgreSQL | 🟢 Alto en contenido | Convertir en práctica con Node |
| MySQL | 🟡 Transferible desde SQL | Repasar diferencias si la empresa lo usa |
| Testing | 🟢 Cubierto en NestJS | Probar handlers e integraciones AWS |
| Seguridad | 🟢 Conceptual | IAM y Secrets Manager son brechas AWS |
| CI/CD | 🟢 Transferible | Crear pipeline para serverless |
| Incidentes/RCA | 🟡 | Preparar una historia real y un laboratorio |
| Git y colaboración | 🟢 Alto | Mostrar PR y code review |

## Qué se transfiere desde tu stack

| Concepto | Tu base | Equivalente o aplicación AWS |
|---|---|---|
| Función serverless | Azure Functions | AWS Lambda |
| API gateway administrado | Azure API Management | Amazon API Gateway |
| Object storage | Azure Blob Storage | Amazon S3 |
| Cola | Azure Service Bus Queue | Amazon SQS |
| Bus de eventos | Event Grid/Service Bus Topics | Amazon EventBridge |
| Base relacional gestionada | Azure SQL/Azure PostgreSQL | Amazon RDS |
| Observabilidad | Azure Monitor/Application Insights | Amazon CloudWatch, X-Ray/OTel |
| Identidad del workload | Managed Identity | IAM role para Lambda |
| Secretos | Key Vault | Secrets Manager/Parameter Store |

La equivalencia acelera el aprendizaje, pero no vuelve intercambiables los productos. Debes conocer semántica, límites, permisos, reintentos y operación de AWS.

## Prioridad P0 — Node.js y JavaScript backend

### Runtime

- Event Loop, microtasks, timers y Worker Pool.
- Operaciones bloqueantes y trabajo CPU-bound.
- Promises, `async`/`await`, manejo de errores y concurrencia controlada.
- Streams y backpressure a nivel práctico.
- ESM vs CommonJS.
- Variables de ambiente, señales y cierre ordenado.
- Dependencias, lockfile, auditoría y versiones LTS.

Node escala bien para I/O siempre que el trabajo por callback sea pequeño; bloquear el Event Loop perjudica throughput y puede convertirse en riesgo de disponibilidad. Repasar [Node.js: no bloquear Event Loop ni Worker Pool](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop).

Material: [[Fundamentos Node y TypeScript]], [[03 - Node.js Event Loop y Modulos]], [[02 - TypeScript Moderno para NestJS]], [[04 - npm pnpm y Gestion de Dependencias]].

### API sin magia de framework

Debes poder explicar aunque utilices NestJS:

- request/response y middleware;
- validación y serialización;
- contratos y versionado;
- códigos HTTP;
- manejo centralizado de errores;
- timeouts y cancelación;
- autenticación/autorización;
- paginación y rate limiting.

Material: [[APIs REST NestJS]], [[01 - DTOs y Class Validator]], [[03 - Exception Filters y Manejo de Errores]], [[05 - Versionado y Documentacion con Swagger]].

## Prioridad P0 — AWS serverless

### Lambda

Aprender:

- handler, evento, contexto y respuesta;
- invocación síncrona, asíncrona y event source mapping;
- timeout, memoria, concurrencia y cold start;
- reutilización del execution environment;
- clientes SDK y conexiones fuera del handler cuando corresponda;
- variables, layers y empaquetado;
- IAM execution role;
- idempotencia y duplicados.

AWS advierte que los event source mappings pueden procesar al menos una vez y producir duplicados, por lo que las funciones deben ser idempotentes: [Lambda best practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html).

### API Gateway

- HTTP API vs REST API a nivel conceptual.
- Routes, stages, integrations y mappings.
- JWT authorizer/Lambda authorizer/API keys según caso.
- Throttling, quotas y límites de payload.
- CORS.
- Respuestas de error consistentes.
- Logs y métricas de acceso.

### S3

- Buckets, keys, metadata y versionado.
- Acceso privado por IAM/bucket policy.
- Presigned URLs para upload/download.
- Multipart upload para archivos grandes.
- Eventos S3 y procesamiento asíncrono.
- Lifecycle y retención.
- No tratar S3 como filesystem POSIX.

### SQS

- Standard vs FIFO.
- At-least-once delivery y orden.
- Visibility timeout.
- Long polling y batch.
- Retrys y `maxReceiveCount`.
- Dead-letter queue y redrive.
- Partial batch response.
- Idempotency key y deduplicación.

En una integración Lambda-SQS, el timeout de la función debe ser menor o igual al visibility timeout, y una DLQ evita que mensajes venenosos degraden continuamente la cola: [configuración SQS para Lambda](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-configure.html).

### EventBridge

- Event bus, event, rule, pattern y target.
- Eventos de dominio vs tareas dirigidas a una cola.
- Contratos versionados.
- Retry policy y DLQ.
- Archive/replay como concepto operativo.
- Diferencia entre EventBridge y SQS.

EventBridge reintenta entregas fallidas y puede enviar eventos no entregados a una DLQ; una DLQ debe configurarse explícitamente para evitar perderlos tras agotar reintentos: [retries de EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rule-retry-policy.html).

### RDS con PostgreSQL

- Subnets/security groups a nivel conceptual.
- Credenciales desde Secrets Manager.
- Migrations fuera del handler de negocio.
- Pooling y riesgo de connection storms.
- RDS Proxy cuando el patrón de concurrencia lo justifique.
- Índices, transacciones y planes de ejecución.
- Backup, Multi-AZ y read replicas a nivel de decisión.

Material: [[POSTGRESQL]], [[01 - Conexion desde NestJS]], [[02 - EXPLAIN ANALYZE]], [[05 - Testing con Postgres]].

### CloudWatch y trazabilidad

- Logs por función y logs estructurados.
- Logs Insights.
- Métricas de Lambda, API Gateway, SQS y RDS.
- Alarmas sobre errores, throttles, latencia, edad de mensajes y DLQ.
- Correlation ID/trace ID.
- X-Ray u OpenTelemetry para trazas.
- Dashboard y runbook asociado a cada alerta.

## Procesamiento asíncrono — modelo de respuesta

```text
API Gateway → Lambda de ingreso
                   │
                   ├── valida y persiste estado inicial en RDS
                   └── publica mensaje en SQS
                                  │
                                  ▼
                         Lambda consumidora
                           │            │
                           │ éxito      └─ fallo → retry → DLQ
                           ▼
                     actualiza RDS
                     emite evento
                     en EventBridge
```

Debes explicar:

- qué operación es transaccional;
- cómo se evita procesar dos veces;
- qué timeout aplica en cada salto;
- cuándo reintentar y cuándo no;
- cómo se inspecciona y redrive una DLQ;
- qué métricas alertan antes de que el usuario reporte el fallo.

## Testing requerido

### Unit tests

- reglas de negocio;
- validadores y mappers;
- handler con puertos mockeados;
- idempotencia y clasificación de errores;
- retryable vs non-retryable.

### Integration tests

- PostgreSQL real con Testcontainers;
- contrato OpenAPI;
- adaptadores AWS con entorno de prueba o integración controlada;
- procesamiento de batch con éxitos y fallos parciales;
- autorización y secretos.

### Evitar

- mockear absolutamente todo y nunca comprobar SQL;
- tests que solo verifican llamadas internas;
- depender de una cuenta AWS compartida sin aislamiento;
- hacer E2E costosos para cada caso de borde.

Material: [[Testing NestJS]], [[01 - Jest y Estructura de Tests]], [[02 - Unit Testing de Providers y Servicios]], [[04 - Testing E2E con Supertest]].

## Seguridad para esta vacante

- IAM least privilege para cada Lambda.
- Secrets Manager/Parameter Store, nunca secretos en variables versionadas.
- Validación de entrada y límites de payload.
- JWT/OAuth 2.0/API keys según el tipo de integración.
- Rotación y auditoría de credenciales.
- Cifrado en tránsito y reposo.
- S3 privado por defecto.
- Protección ante SQL injection, SSRF y abuso de endpoints.
- Dependencias auditadas.
- Datos sensibles fuera de logs.

Material: [[Auth y Seguridad NestJS]], [[02_📚 Estudio/06_🐈‍⬛ NestJS/13_ Senior NestJS/03 - Seguridad Senior|Seguridad Senior NestJS]], [[05 - Helmet CORS y Seguridad HTTP]], [[06 - OAuth2 y Login Social]].

## Proyecto mínimo para demostrar el stack

Construir una integración de órdenes o inventario:

1. API Gateway recibe `POST /orders`.
2. Lambda Node valida el payload y contrato.
3. PostgreSQL en RDS almacena orden e idempotency key.
4. SQS desacopla procesamiento.
5. Lambda consumidora procesa batch y soporta partial failures.
6. DLQ recibe mensajes no recuperables.
7. EventBridge publica `OrderProcessed`.
8. S3 almacena comprobantes mediante presigned URL.
9. CloudWatch registra logs estructurados, métricas y alarmas.
10. OpenAPI documenta la API.
11. Pipeline ejecuta tests y despliega.

Usar AWS SAM o CDK como P1 para definir infraestructura. No es necesario aprender varias herramientas IaC a la vez.

## Sprint de preparación — diez días

| Día | Foco | Entregable |
|---|---|---|
| 1 | JavaScript/Node avanzado | ejercicios async y diagnóstico de bloqueo |
| 2 | API REST + OpenAPI | handler/API validada y documentada |
| 3 | Lambda + IAM | función desplegada con permisos mínimos |
| 4 | API Gateway + auth | endpoint, errores y throttling |
| 5 | S3 | presigned upload/download |
| 6 | SQS + Lambda | batch, idempotencia, visibility y DLQ |
| 7 | EventBridge | evento, regla, target, retry y DLQ |
| 8 | RDS PostgreSQL | persistencia, índice y test de integración |
| 9 | CloudWatch | queries, dashboard, alarms y trazabilidad |
| 10 | CI/CD + simulación | deploy reproducible y entrevista técnica |

No esperar diez días para postular; ejecutar el sprint mientras la candidatura avanza.

## Preguntas probables

### Node.js

- ¿Qué bloquea el Event Loop?
- ¿Cómo limitas concurrencia al llamar servicios externos?
- ¿Qué diferencia existe entre `Promise.all` y procesamiento secuencial?
- ¿Cómo manejas errores asíncronos y shutdown?
- ¿Por qué usarías TypeScript aunque el anuncio diga JavaScript?

### AWS

- ¿Cuándo usar SQS y cuándo EventBridge?
- ¿Cómo haces idempotente una Lambda?
- ¿Qué ocurre si una Lambda falla procesando un batch SQS?
- ¿Cómo defines visibility timeout y DLQ?
- ¿Cómo evitas demasiadas conexiones de Lambda hacia RDS?
- ¿Cómo asegurarías un bucket S3?
- ¿Qué observarías en CloudWatch ante una cola atrasada?

### Diseño

- Diseña una integración con un proveedor que falla intermitentemente.
- ¿Qué errores deben reintentarse?
- ¿Cómo evitas duplicados cuando el proveedor hace timeout después de procesar?
- ¿Cómo versionas eventos y APIs?
- ¿Cómo haces root cause analysis de una orden perdida?

### SQL

- ¿Cómo eliges un índice?
- ¿Cómo diagnosticas una consulta lenta?
- ¿Qué transacción protege la idempotencia?
- ¿Cuándo usarías PostgreSQL frente a NoSQL?

## Palabras clave para el CV

`Node.js`, `JavaScript`, `TypeScript`, `REST APIs`, `OpenAPI/Swagger`, `PostgreSQL`, `SQL`, `Git`, `unit testing`, `integration testing`, `event-driven architecture`, `idempotency`, `retry`, `timeouts`, `DLQ`, `logging`, `metrics`, `distributed tracing`, `CI/CD`, `root cause analysis`.

Añadir `AWS Lambda`, `API Gateway`, `S3`, `SQS`, `EventBridge`, `RDS` y `CloudWatch` solo al nivel que puedas demostrar. Diferenciar “experiencia” de “laboratorio/proyecto”.

## Preguntas antes de aceptar una entrevista

1. ¿Node.js se usa con NestJS, Express, Fastify o handlers Lambda puros?
2. ¿JavaScript o TypeScript?
3. ¿Cuánto del sistema es serverless y cuánto corre en containers?
4. ¿Cómo gestionan infraestructura como código?
5. ¿Existe on-call y con qué frecuencia?
6. ¿Qué motor usa RDS: PostgreSQL o MySQL?
7. ¿Qué herramientas utilizan para tracing y CI/CD?
8. ¿La presencialidad es diaria y en qué sede?

## Decisión final

La vacante merece postulación si el rol backend y el salario te interesan, pero debe clasificarse como **rama secundaria Node/AWS**, no como validación de tu stack Angular/.NET/Azure. Si recibes entrevista, el sprint AWS pasa a prioridad alta temporal. Si no, basta el laboratorio pequeño hasta que AWS se repita en nuevas vacantes.
