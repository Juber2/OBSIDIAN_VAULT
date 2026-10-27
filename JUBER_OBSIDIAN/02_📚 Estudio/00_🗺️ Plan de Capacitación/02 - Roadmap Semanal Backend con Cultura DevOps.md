---
tags:
  - Estudio
  - Roadmap
  - Capacitación
Creado: 2026-08-31
Actualizado: 2026-08-31
Inicio: 2026-08-31
Fin: 2026-12-27
Relacionado:
  - 01 - Mapa de Capacitacion (set-dic 2026)
  - 09 - Perfil Objetivo - Puestos con Peso
---

# Roadmap semanal — Backend con cultura DevOps

17 semanas, del **31 de agosto al 27 de diciembre de 2026**. Ejecución de [[01 - Mapa de Capacitacion (set-dic 2026)]], calibrada por los 5 avisos que definen el objetivo.

## El objetivo

**Desarrollador Backend que diseña, prueba, despliega y opera.** No DevOps Engineer, no arquitecto todavía.

Los 5 avisos coinciden en 4 o 5 de 5 sobre lo mismo: **API + arquitectura + entrega + operación**. Ninguno de esos cuatro es un lenguaje.

## El entregable central: un sistema, dos servicios, dos nubes

Todo el roadmap converge en una sola cosa construida:

```
        [ Servicio .NET ]  ←── mensajería ──→  [ Servicio Java/Spring ]
              │                                        │
        Azure: App Service · Key Vault           AWS: Lambda · SQS · RDS
              │                                        │
              └────── observabilidad correlacionada ───┘
                              │
                    CI/CD con rollback
```

Por qué así y no un solo servicio en un solo stack:

| Decisión | Qué credencial produce |
|---|---|
| **Base en .NET** | Sale rápido: sos experto, cero curva. Libera semanas para lo que no sabés |
| **Un servicio en Java/Spring** | Java deja de ser aspiracional. Dos semanas alcanzan apoyándote en las equivalencias con .NET |
| **Azure + AWS** | **Multi-cloud** es raro y bien pagado. Stefanini lo exige junto; Cuidafarma pide AWS; ya tenés Azure real |
| **Dos servicios que se hablan** | Es *la* demostración de arquitectura, y responde la pregunta 52 de tu banco: *"¿cómo integrás un servicio Java con uno .NET?"* |

> [!important] Reestructurado el 31/08/2026
> La versión anterior ponía un mes entero en Java y AWS como "vocabulario". Con el criterio de [[09 - Perfil Objetivo - Puestos con Peso]] eso estaba invertido: **AWS lo piden 3 de 5 avisos (dos como experiencia práctica) y Java 3 de 5, pero Java solo desbloquea una vacante alcanzable.** AWS sube a 2 semanas y media prácticas; Java baja a dos, integrado al mismo sistema.

## Qué sobrevive de tu roadmap DevOps (nov 2025)

| Del roadmap viejo | ¿Sigue? |
|---|---|
| Git avanzado, Docker, Azure, CI/CD, OWASP, microservicios, Linux | ✅ **Sí** — están en 3 a 5 de los avisos |
| Kubernetes | ✅ Reducido a fundamentos y diagnóstico |
| Monitoring | ✅ Vía App Insights y CloudWatch |
| Blue-green y canary | ✅ Como rollback |
| Terraform | ⚠️ Solo el concepto de IaC |
| ArgoCD, Prometheus/Grafana, **AZ-104 / AZ-400**, networking, HA/DR, FinOps | ❌ Fuera — otro rol, y consumen el tiempo del laboratorio |

Se agrega lo que ese roadmap no tenía y el mercado sí pide: **arquitectura aplicada, testing, Java, mensajería con idempotencia y AWS**.

## Estructura de cada semana

```
🎯 Objetivo    → qué debés poder hacer al terminar
📚 Estudiar    → material del vault
🔨 Construir   → lo que produce evidencia
✅ Entregable  → cómo sabés que la semana cerró
```

Horario diario en [[03 - Horario Diario Realista (8am-8pm)]] · ~24 h/semana.

---

# MES 1 — Arquitectura y núcleo

*Arquitectura primero porque está en 5 de 5 avisos, no depende de nada, y sin ella el laboratorio sale mal.*

## 📅 Semana 1 (31 ago – 6 sep) — Arquitectura y patrones

- 🎯 Defender cualquier pregunta de diseño sin leer
- 📚 Capas y regla de dependencia · hexagonal (puertos y adaptadores) · SAGA · Cache-Aside
- 📚 Patrones GoF: Singleton, Factory, Repository, Strategy, Circuit Breaker
- 🔨 Elegir 5 patrones y escribir para cada uno: definición, ejemplo propio, cuándo NO usarlo
- ✅ Explicar la diferencia entre capas y hexagonal, y cuándo usar cada una, en voz alta

## 📅 Semana 2 (7 – 13 sep) — DDD, CQRS y diseño del sistema

> Un arquitecto diseña antes de codear. Esta semana **no se escribe código**.

- 🎯 Tener el sistema completo diseñado en papel antes de la primera línea
- 📚 DDD: bounded contexts, agregados, eventos de dominio vs integración · CQRS sin sobreingeniería
- 🔨 **Diagrama de arquitectura** del sistema completo en Draw.io: servicios, colas, bases, nubes
- 🔨 **Primer ADR**: por qué dos servicios y no uno; qué alternativa descartaste
- ✅ Un diagrama y un ADR que podés defender — la credencial que piden TCS y Stefanini

## 📅 Semana 3 (14 – 20 sep) — La API base y los datos

- 🎯 API funcionando con dominio separado y decisiones de datos tomadas con criterio
- 📚 Separación de capas, DTOs, ProblemDetails
- 📚 **Contrato de API completo** (Cuidafarma lo pide punto por punto): versionado, códigos HTTP, validaciones, manejo de errores, **paginación**, autenticación y **API Keys**
- 📚 **Datos** — 3 de 5 avisos: índices y planes de ejecución, transacciones y **concurrencia**, **ACID vs BASE**, cuándo un documento gana a una tabla (**NoSQL**), y qué cambia entre SQL Server, PostgreSQL y **MySQL**
- 📚 **SQL avanzado** — el delta sobre tus 5 años es chico y diferencia: window functions, CTEs, y **leer un plan de ejecución** en serio (scans vs seeks, estimado vs real, parameter sniffing)
- 🔨 API .NET: dominio, casos de uso, EF Core, validación, errores, **OpenAPI**, endpoint paginado
- ✅ `docker compose up` levanta API y base
- ✅ Podés justificar cada índice del modelo y decir cuándo elegirías NoSQL

> [!note] Por qué datos va acá y no en una semana propia
> Ya tenés 5 años de SQL y 170 notas entre SQL Server y PostgreSQL. Lo que falta no es aprender bases de datos: es **ACID vs BASE, NoSQL y planes de ejecución**, y se fija mejor decidiéndolo sobre un modelo real que estudiándolo suelto.

## 📅 Semana 4 (21 – 27 sep) — Testing en profundidad

> El hueco más visible: 3 de 5 avisos lo piden y tu CV no lo menciona **ni una vez**.

- 🎯 Saber qué probar con mocks, qué con integración, y qué no probar
- 📚 xUnit, TDD, `WebApplicationFactory`, Testcontainers
- 🔨 Tests unitarios del dominio + integración sobre endpoints con base real
- ✅ Explicar tu estrategia de testing en 2 minutos con el repo abierto

## 📅 Semana 5 (28 sep – 4 oct) — Entrega: Docker, Git, Scrum, Postman, Sonar

- 🎯 Cerrar la parte de "cultura DevOps" que sí piden los avisos
- 📚 Docker multi-stage y capas · Git avanzado (rebase, conflictos, branching)
- 📚 **Agile/Scrum**: ceremonias, roles, Scrum vs Kanban — medio día, es vocabulario
- 📚 **Postman/Newman** · **SonarQube** — horas, no semanas
- 📚 **Code review y pull requests** — Cuidafarma lo pide explícito y es 4 de 5 en Git/colaboración: qué mirás en una revisión y cómo dejás un comentario que no bloquea al equipo
- 🔨 Dockerfile optimizado y medido · collection de Postman corriendo con Newman
- ✅ Imagen reducida con número antes/después — es una línea de CV

---

# MES 2 — Seguridad, Azure y mensajería

*Convierte cuatro capacidades de "sé la teoría" en "está desplegado".*

## 📅 Semana 6 (5 – 11 oct) — Seguridad

- 🎯 Responder qué diferencia OAuth2, OIDC y JWT, y proteger de verdad
- 📚 [[04 - Entra ID y Seguridad de APIs]] · OWASP Top 10 en [[03 - Seguridad Backend Java]]
- 🔨 Entra ID protegiendo la API · policies por scope · **autorización por recurso**
- ✅ Token sin scope → 403, **y** un usuario no accede al recurso de otro. Probar ambos

## 📅 Semana 7 (12 – 18 oct) — Azure: despliegue y secretos

- 🎯 Sistema desplegado sin una sola credencial guardada
- 📚 [[01 - Key Vault y Managed Identity]] · [[05 - Azure SQL y Blob Storage]]
- 🔨 ACR + App Service · Managed Identity · Key Vault · Azure SQL sin password
- ✅ URL pública funcionando · cero secretos en repo, pipeline y configuración

## 📅 Semana 8 (19 – 25 oct) — Mensajería e idempotencia

- 🎯 Responder la pregunta que más se repite en los avisos, con un caso propio
- 📚 [[02 - Service Bus y Mensajeria]] · Outbox
- 🔨 Evento a Service Bus · consumidor idempotente · DLQ · **Outbox**
- ✅ **Publicar el mismo mensaje dos veces y verificar un solo efecto**

## 📅 Semana 9 (26 oct – 1 nov) — Observabilidad

- 🎯 Diagnosticar sin adivinar
- 📚 [[03 - Application Insights y Azure Monitor]] · KQL · percentiles
- 📚 **Redes y latencia** — lo pide Stefanini y es la causa #1 de "la app anda lenta": DNS, TLS y su handshake, balanceo de carga, timeouts y reintentos, connection pooling, y por qué una llamada entre regiones cuesta 10 veces más que una local
- 🔨 Logs estructurados · `operation_Id` propagado · dashboard con p95 y profundidad de cola · una alerta accionable
- ✅ Una consulta KQL reconstruye una operación de punta a punta

---

# MES 3 — AWS y el servicio Java

*El mes que cierra los dos huecos que hoy están en cero.*

## 📅 Semana 10 (2 – 8 nov) — AWS: fundamentos y primera Lambda

> 🔴 **Cero contenido en el vault.** Las 17 apariciones de "Lambda" son de Java.

- 🎯 Moverte en AWS con criterio, no siguiendo un tutorial
- 📚 Modelo de servicios · **IAM y mínimo privilegio** · regiones · free tier · costos
- 📚 Mapeo servicio por servicio contra Azure — lo conceptual ya lo sabés
- 🔨 Cuenta free tier · primera **Lambda en Node.js** detrás de **API Gateway**
- ✅ Un endpoint en AWS que responde, y saber explicar el rol de IAM que usa

## 📅 Semana 11 (9 – 15 nov) — AWS serverless completo

- 🎯 Cubrir el stack exacto que pide Cuidafarma
- 📚 SQS · EventBridge · RDS · S3 · CloudWatch
- 🔨 Flujo completo: API Gateway → Lambda → **SQS** → Lambda consumidora → **RDS**
- 🔨 **DLQ** en la cola · S3 para un archivo · CloudWatch con métrica y alarma
- 📚 **Costos cloud** — qué se paga en Lambda (invocaciones y duración), RDS (siempre encendida), S3 y salida de datos. Poder responder *"¿cuánto costaría esto con mil usuarios?"* distingue de inmediato, y casi ningún desarrollador sabe hacerlo
- ✅ Estimar el costo mensual de tu propio sistema, con los números de la calculadora
- 🔨 Probar el mensaje duplicado, igual que en la semana 8
- ✅ Diagrama de 6 servicios de AWS explicable uno por uno

## 📅 Semana 12 (16 – 22 nov) — El servicio Java/Spring

- 🎯 Que Java deje de ser aspiracional
- 📚 [[06 - Ruta Java Spring Boot como segundo eje backend]] — apoyate en la tabla de equivalencias .NET ↔ Java
- 📚 Spring Core, Boot, Data JPA, Security, JUnit + Mockito · **Maven** (Coforge lo pide explícito)
- 📚 **Quarkus** — 2 de 5 avisos (TCS y Coforge). Dos días desde Spring: por qué procesa en build-time, cuándo lo elegirías, GraalVM. Ya tenés 9 notas escritas
- 🔨 Segundo servicio del sistema en **Spring Boot**: API REST, JPA, seguridad JWT, tests
- ✅ Servicio Java desplegado, con tests, en el mismo sistema

## 📅 Semana 13 (23 – 29 nov) — Integración políglota y CI/CD

- 🎯 Que los dos servicios se hablen, y que todo despliegue solo
- 📚 Contratos entre stacks · versionado de eventos · [[GitHub Actions CI CD]] · Kubernetes 1-6
- 📚 **APIM / API Gateway** — [[04 - APIM y Application Gateway]]: policies, rate limiting y validación de token en el borde
- 📚 **DevSecOps** (Stefanini) — que Sonar y el escaneo de dependencias corran **en el pipeline**, no a mano
- 📚 **OpenShift** en una línea: es Kubernetes de Red Hat con routes y capa propia; lo que sabés de K8s transfiere
- 🔨 El servicio Java consume los eventos del .NET por mensajería
- 🔨 Pipeline: build → test → **Sonar** → imagen → deploy → smoke test → swap
- 🔨 Rollback probado
- ✅ Un push despliega solo · revertís en un comando · **podés explicar cómo conviven dos stacks**

## 📅 Semana 14 (30 nov – 6 dic) — Troubleshooting, ADRs y cierre

- 🎯 Diagnosticar bajo presión y dejar el proyecto defendible
- 🔨 **Romper el sistema a propósito**: tirar la base, vencer un token, llenar la DLQ, meter un N+1, dejar un pod en `CrashLoopBackOff`
- 🔨 **2-3 ADR más** con las alternativas descartadas · diagrama final
- 🔨 **Discurso de IA en el desarrollo** (2 de 5 avisos): qué revisás antes de aceptar código generado, qué no delegás, un caso en que detectaste algo mal
- 📚 Angular, repaso corto — [[03 - Ruta de repaso Angular para entrevistas]]
- ✅ Un mini-postmortem por cada falla provocada · README, diagrama y ADRs completos

### 🔺 Diseño de sistemas — la habilidad que más pesa en entrevista senior

Los cuatro puestos con peso hacen una pregunta de diseño abierta (*"diseñá un sistema de reservas"*). Tenés todas las piezas; falta el **método** para armarlas en 40 minutos frente a alguien. Practicá este guion hasta que salga solo:

```
1. Preguntar antes de dibujar   → ¿cuántos usuarios? ¿lecturas o escrituras?
                                   ¿qué puede fallar sin que sea grave?
2. Estimar en voz alta          → órdenes de magnitud, no exactitud
3. Dibujar el camino feliz      → cliente → API → datos. Simple primero
4. Romperlo vos mismo           → ¿dónde está el cuello de botella?
                                   ¿qué pasa si esto se cae?
5. Agregar solo lo que resuelve → caché, cola, réplica — cada pieza con su motivo
6. Decir qué NO harías          → y por qué. Acá es donde suena senior
```

**El error más común:** empezar a dibujar cajas antes de preguntar nada, y agregar Kafka y Redis "porque escalan". El paso 6 es el que más distingue: nombrar la sobreingeniería que evitaste.

- 🔨 Diseñar en voz alta y cronometrado: un acortador de URLs, un sistema de reservas, un feed de notificaciones
- ✅ Sostener 40 minutos de diseño abierto sin quedarte en blanco ni sobrediseñar

> [!note] Inglés, transversal desde la semana 10
> 15 minutos del bloque 💼: leer documentación de AWS en inglés. Desde la semana 12, ensayar el pitch y 3 respuestas técnicas en voz alta.

---

# MES 4 — Salida al mercado

## 📅 Semana 15 (7 – 13 dic) — CV, perfil y primeras postulaciones

- 📚 [[24 - Guia de Adaptacion del CV (.NET y Java)]]
- 🔨 Dos versiones de CV · LinkedIn con keywords · GitHub con el sistema fijado y README serio
- ✅ Enviado a Cuidafarma y a las vacantes 🟢 (Entelgy, TCS .NET, TSOFT)

## 📅 Semana 16 (14 – 20 dic) — Simulacros y postulación activa

- 🔨 Las 82 preguntas de [[20 - Banco de Preguntas de las 14 Vacantes]] hasta que el Nivel 1 salga sin pensar
- 🔨 Dos simulacros completos: uno .NET, uno Java
- 🔨 **Dos ejercicios de diseño de sistemas** con el guion de la semana 14, cronometrados en 40 minutos
- ✅ 10+ postulaciones enviadas con CV adaptado

## 📅 Semana 17 (21 – 27 dic) — Semana baja

- 🔨 Repaso ligero con [[05 - Repaso Espaciado (para no olvidar lo estudiado)]]
- ✅ Descansar

> [!warning] Timing del mercado peruano
> Fin de diciembre los procesos se congelan. **El pico de contratación es enero-marzo.** Por eso el CV y el portafolio están listos en la semana 15, no el 31: diciembre deja procesos abiertos, el empuje real es la primera semana de enero.

---

# Verificación de cobertura

Cruce contra los 5 avisos de [[01 - Mapa de Capacitacion (set-dic 2026)]]:

| Capacidad | Piden | Semanas |
|---|:-:|---|
| APIs REST + OpenAPI | 5/5 | S3 |
| Clean Architecture / SOLID / DDD | 5/5 | S1, S2, S14 |
| Microservicios | 4/5 | S12, S13 |
| Cloud práctico (Azure + AWS) | 4/5 | S7, S10, S11, S13 |
| CI/CD | 4/5 | S13 |
| Observabilidad | 4/5 | S9 |
| Event-driven, idempotencia, DLQ | 4/5 | S8, S11 |
| Git y colaboración | 4/5 | S5 |
| Seguridad OAuth2 / JWT / OWASP | 3/5 | S6 |
| Testing (unit, integración, TDD) | 3/5 | S4, S12 |
| Docker y Kubernetes | 3/5 | S5, S13 |
| Java + Spring Boot | 3/5 | S12, S13 |
| Bases relacionales y SQL | 3/5 | S3, S7 |
| Node.js desplegado | 2/5 | S10, S11 |
| IA en el desarrollo | 2/5 | S14 |
| SonarQube | 2/5 | S5, S13 |
| Diagramas de arquitectura | 2/5 | S2, S14 |
| Angular | 1/5 | S14 |

**Sin semana asignada, a propósito:** Oracle (0 de 5), Python (2 de 5, no alcanza el tiempo), microfrontends, FinOps, ELK/Dynatrace. Ver el bloque "fuera del plan" del mapa.

> [!tip] Repetir este cruce con cada aviso nuevo
> Si un requisito llega a 2 de 5, necesita semana asignada — no una mención.

# Los 4 entregables que deciden todo

| # | Entregable | Cierra |
|---|---|---|
| 1 | **Sistema de dos servicios en dos nubes, desplegado y con repo público** | Java sin experiencia, AWS, testing, seguridad, mensajería, observabilidad, microservicios |
| 2 | **CV en dos versiones** (.NET y Java honesta) | Las 9 acciones de [[23 - Mi CV frente a las 14 Vacantes]] |
| 3 | **Inventario de ejemplos propios completo** | Las 91 veces que las notas piden "ejemplo propio" |
| 4 | **Nivel 1 del banco de preguntas fluido** | Las 12 preguntas que hacen 4+ vacantes |

# Revisión mensual

Último domingo de cada mes, una sola pregunta: **¿el sistema avanzó?**

Si dos meses seguidos avanzó el estudio pero no el sistema, el roadmap está fallando aunque los checks estén marcados. La corrección es cortar temas nuevos y construir — igual que el roadmap de DevOps de 2025, que tenía 40 semanas de contenido y quedó sin terminar porque el volumen le ganó a la ejecución.
