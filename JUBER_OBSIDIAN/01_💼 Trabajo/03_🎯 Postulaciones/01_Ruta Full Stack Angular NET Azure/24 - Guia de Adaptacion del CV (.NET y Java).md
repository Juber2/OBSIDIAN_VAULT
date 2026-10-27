---
tags:
  - Postulaciones
  - CV
Creado: 2026-08-31
Relacionado:
  - 23 - Mi CV frente a las 14 Vacantes
  - 13 - Sistema de postulación por vacante
---

# Guía de adaptación del CV (.NET y Java)

Cómo reescribir el CV para que refleje lo que **realmente hiciste** y no lo comunica hoy, más la versión orientada a Java que señala capacidad sin afirmar experiencia laboral inexistente.

## La regla que separa adaptar de inventar

| ✅ Legítimo | ❌ No |
|---|---|
| Escribir algo que hiciste y omitiste | Poner una tecnología en un empleo donde no la usaste |
| Usar el nombre técnico correcto de lo que hacías | Inflar el alcance de un proyecto real |
| Detallar un proyecto propio real | Inventar un proyecto |
| Reordenar para destacar lo relevante | Cambiar fechas o duraciones |
| Listar tecnologías de proyectos personales, **etiquetadas como tales** | Listarlas mezcladas con la experiencia laboral |

> La prueba: **¿puedo sostener 10 minutos de repreguntas técnicas sobre esta línea?** Si no, sale.

## Convención de esta guía

- ✅ **Ya lo hiciste** — está en tu CV, solo mal escrito. Usalo tal cual.
- ❓ **Confirmá** — es muy probable que lo hicieras, pero solo vos sabés. Si es sí, usá la redacción; si es no, borralo.

---

# Parte 1 — Lo que casi seguro hiciste y no está en el CV

Estas ausencias cuestan filtros. Revisá cada una:

| ❓ Confirmá si… | Lo piden | Si es sí, agregalo en |
|---|---:|---|
| Escribiste **pruebas unitarias** (xUnit, NUnit, MSTest) aunque sea en algunos módulos | 6/14 | Pluxee o el sistema de capacitación |
| Documentaste APIs con **Swagger/OpenAPI** | 4/14 | Pluxee, Trener |
| Implementaste **autenticación/autorización** (JWT, Identity, roles) | 7/14 | Cualquier sistema con login — casi seguro Trener y Pluxee |
| Trabajás con **ramas, pull requests y code review** | 6/14 | Pluxee (si usás Azure Repos con PRs) |
| Trabajás en **Scrum/sprints**, dailies, refinamiento | 7/14 | Pluxee, NTT DATA (mencionás Jira) |
| Usaste **Postman** para probar o documentar APIs | 3/14 | Cualquiera |
| Manejaste **caché** (memoria, Redis) en algún módulo | 2/14 | Si aplica |
| Consumiste o expusiste **servicios SOAP/XML** | 1 (Synopsis) | NTT DATA / Trener, si tocaste sistemas legacy |
| Trabajaste con **colas o procesos asíncronos** (Hangfire, jobs, timers) | 4/14 | Los batch/ETL de Pluxee podrían calificar |
| Hiciste **rollback** de un despliegue alguna vez | 5/14 | Es una historia de entrevista, no una línea de CV |

> [!tip] La más importante
> **Testing** es la ausencia más visible: aparece en 6 vacantes y tu CV no la menciona ni una vez. Si escribiste aunque sea tests de algunos servicios, tiene que estar.

---

# Parte 2 — Reescritura, línea por línea

## Perfil profesional

**Hoy dice:** *"Analista desarrollador de software con más de 4 años de experiencia..."*

**Problema:** el título no coincide con ninguna búsqueda. Nadie busca "Analista desarrollador".

**Versión .NET:**

> Desarrollador Backend .NET con más de 4 años construyendo APIs REST y sistemas empresariales con C#, ASP.NET Core, Entity Framework y SQL Server. Experiencia en Clean Architecture, CQRS, optimización de base de datos, integración con servicios de terceros y despliegue continuo con Docker y Azure DevOps. Enfocado en backend, arquitectura y entornos cloud.

Cambios: título buscable, "APIs REST" explícito, CQRS al frente, "integración con terceros" (10/14 vacantes), cloud al cierre.

## Pluxee / Experis — tu experiencia más vendible

| Hoy | Reescrito |
|---|---|
| *"Desarrollo de APIs bajo Clean Architecture y principios SOLID"* | ✅ "Desarrollo de **APIs REST** en **ASP.NET Core** aplicando **Clean Architecture** y SOLID ❓ *(+ documentadas con Swagger/OpenAPI)* ❓ *(+ con pruebas unitarias sobre la capa de aplicación)*" |
| *"Integración de pagos mediante QR (Izipay)"* | ✅ "**Integración con el proveedor externo de pagos Izipay (QR)**: consumo de API de terceros, manejo de errores y validación de estados de transacción ❓ *(+ control de duplicidad para evitar cobros repetidos)*" |
| *"Migración de módulos legacy a arquitectura moderna de back office"* | ✅ "**Modernización incremental** de módulos legacy hacia una arquitectura mantenible, conviviendo con el sistema existente sin interrumpir la operación" |
| *"procesos batch y ETLs para manejo de datos masivos"* | ✅ "Desarrollo y optimización de **procesos batch y ETL** para procesamiento de **altos volúmenes de datos**, con control de errores y reprocesamiento" |
| *"Contenerización de aplicaciones con Docker"* | ✅ "**Contenerización con Docker** para entornos reproducibles entre desarrollo y producción" |
| *"pipelines CI/CD en Azure DevOps utilizando YAML"* | ✅ "Implementación de **pipelines CI/CD en Azure DevOps (YAML)** con build y despliegue automático por commit ❓ *(+ ejecución de pruebas en el pipeline)*" |
| *"migrando de un esquema manual en IIS a despliegues continuos"* | ✅ "**Automatización del despliegue**: migración de un proceso manual en IIS a despliegue continuo, **reduciendo tiempo y errores de publicación**" |

❓ **Agregar si aplica:** "Trabajo bajo metodología **Scrum**, con control de versiones en Git, **pull requests y code review**."

## Colegio Trener — donde está tu mejor historia de performance

| Hoy | Reescrito |
|---|---|
| *"Optimización de consultas y procedimientos almacenados... reduciendo tiempos de respuesta en módulos críticos"* | ✅ "**Optimización de consultas y stored procedures en SQL Server** mediante análisis de **planes de ejecución** e **índices**, reduciendo tiempos de respuesta en módulos críticos **de X a Y segundos**" ← poné el número si lo tenés |
| *"Diseño e implementación de APIs para integración con aplicaciones móviles"* | ✅ "Diseño e implementación de **APIs REST** consumidas por aplicaciones móviles ❓ *(+ con autenticación mediante JWT)*" |
| *"Implementación completa de sistemas de admisión y suscripción (end-to-end)"* | ✅ "Implementación **end-to-end** de los sistemas de admisión y suscripción: modelado de datos, backend, frontend y puesta en producción" |
| *"integrados con SQL Server y FoxPro"* | ✅ "Integración de sistemas web con **SQL Server** y bases legacy **FoxPro**" ← "legacy" es palabra clave para Periferia |

## NTT DATA — desenterrar BBVA

| Hoy | Reescrito |
|---|---|
| *"Automatización de procesos mediante Power Automate y macros (proyecto BBVA)"* | ✅ "Automatización de procesos para **BBVA (sector bancario)**, reduciendo tareas manuales del equipo operativo" ← el sector va adelante, no entre paréntesis |
| *"Migración de aplicaciones y bases de datos a Azure"* | ✅ "**Migración de aplicaciones y bases de datos a Azure**, incluyendo configuración de entornos y validación posterior" |
| *"Optimización de procedimientos almacenados y generación de reportes de alto volumen"* | ✅ "Optimización de stored procedures y generación de **reportes de alto volumen** sobre SQL Server" |

> **Sector financiero:** con BBVA aquí + Izipay en Pluxee, podés afirmar honestamente *"experiencia en proyectos del sector financiero y de medios de pago"* en el perfil cuando postules a [[Synopsis - Backend Senior Financiero]], [[Java Senior BIAN OpenShift Azure APIM]] o Coforge/BCP.

## Sistema de Capacitación (freelance) — tu mejor carta de arquitectura

Ya está bien escrito. Solo agregar lo que falte:

- ❓ Autenticación/autorización implementada (JWT, roles)
- ❓ Pruebas unitarias sobre handlers de MediatR
- ❓ Swagger para documentar la API
- ❓ Si está desplegado: dónde y cómo
- ✅ Cambiar *"Implementación de registro de logs para monitoreo"* → "**Logging estructurado** para monitoreo y diagnóstico"

---

# Parte 3 — La versión Java honesta

El objetivo: que un reclutador que busca Java te encuentre y te considere, **sin que un entrevistador descubra una afirmación falsa**.

## Movimiento clave: dividir la sección de conocimientos en dos niveles

Esto es lo que resuelve el problema. Los filtros automáticos leen las palabras; una persona lee el encabezado y entiende el nivel. Ninguna de las dos cosas es engañada:

```
CONOCIMIENTOS TÉCNICOS

Backend — experiencia profesional
   .NET (C#): ASP.NET Core, Web API, MVC, Entity Framework Core
   APIs REST, Clean Architecture, CQRS, MediatR, SOLID
   SQL Server, PostgreSQL, MySQL, Oracle

Backend — proyectos propios y formación
   Java 17: Spring Boot, Spring Data JPA, Spring Security
   Maven, JUnit, Hibernate
   Android nativo (Java / Kotlin)
```

**Por qué funciona:** el ATS indexa "Java", "Spring Boot", "Spring Security", "Hibernate", "JUnit". El reclutador ve el encabezado *"proyectos propios y formación"* y sabe exactamente qué está leyendo. No hay nada que se caiga después.

## Perfil profesional, versión Java

> Desarrollador Backend con más de 4 años de experiencia en sistemas empresariales, APIs REST y bases de datos relacionales, con base sólida en **arquitectura (Clean Architecture, CQRS), integración de servicios y despliegue continuo**. Mi experiencia profesional es en .NET/C#; actualmente desarrollo en **Java 17 y Spring Boot** en proyectos propios, aplicando los mismos principios de arquitectura, testing y cloud. Busco un rol backend donde aportar experiencia en diseño de APIs y evolucionar en el ecosistema Java.

Es honesto, dice "Java" tres veces, y explica **por qué tu experiencia .NET es transferible** — que es el argumento que realmente te tiene que comprar el entrevistador.

## Elevar la sección de proyectos

Hoy tus proyectos Java están en una línea suelta dentro de *"Otros proyectos"*. Deben ser una sección propia y detallada:

```
PROYECTOS PROPIOS

API de Órdenes — Java 17 / Spring Boot            [github.com/tu-usuario/...]
   API REST con Spring Boot, Spring Data JPA y PostgreSQL, aplicando
   arquitectura por capas y separación de dominio e infraestructura.
   Seguridad con Spring Security y JWT, autorización por rol y por recurso.
   Mensajería asíncrona con manejo de idempotencia y dead letter queue.
   Pruebas unitarias (JUnit, Mockito) e integración con Testcontainers.
   Contenerizado con Docker y desplegado en Azure, con logging estructurado
   y trazabilidad entre servicios.

[Tus proyectos freelance en Java, con el mismo nivel de detalle]

Aplicaciones móviles Android — Java / Kotlin
   Apps nativas para control de asistencia y reproducción multimedia,
   con Jetpack Compose y consumo de APIs REST.
```

> [!important] El primero es el que te falta construir
> Ese proyecto es exactamente el [[06 - Laboratorio Integrado - API de Ordenes]] de las 3 semanas. Si lo hacés en **Spring Boot en lugar de .NET**, en tres semanas tenés evidencia Java real, desplegada, con repo abierto — y cubrís de una sola vez los cinco huecos del CV: **testing, seguridad, mensajería, observabilidad y microservicios**.

## Formación

```
FORMACIÓN COMPLEMENTARIA

Java y Spring Boot (en curso, 2026)
   Spring Boot, Spring Data JPA, Spring Security, JUnit, Maven.
   Arquitectura hexagonal, patrones de diseño, mensajería y microservicios.

Cloud y DevOps
   Azure (App Service, Key Vault, Service Bus, Application Insights),
   Docker, Kubernetes, CI/CD.
```

"En curso" es la palabra que lo hace honesto y a la vez muestra iniciativa.

---

# Parte 4 — Palabras clave por vacante

Adaptá el bloque de conocimientos según a dónde postulás (15 minutos, según [[13 - Sistema de postulación por vacante]]):

| Vacante | Palabras que deben aparecer |
|---|---|
| [[Entelgy - Backend NET Azure]] | ASP.NET Core, EF Core, **Dapper**, SQL Server, Azure App Service, Key Vault, Service Bus, Application Insights, Entra ID, GitHub Actions, Clean Architecture, CQRS |
| [[TCS - NET Azure Android]] | .NET, Azure, **Android**, APIM, Azure Functions, Blob Storage, Linux, Scrum, Jira |
| [[TSOFT - NET Angular Oracle]] | .NET, Angular, **Oracle, PL/SQL**, stored procedures, planes de ejecución, alta concurrencia |
| [[Periferia - Migracion VB6 NET 10 WinForms]] | **Visual Basic, WinForms, WPF, Web Forms**, migración legacy, FoxPro |
| [[Synopsis - Backend Senior Financiero]] | **sector financiero, medios de pago (Izipay), BBVA**, OAuth2, integración de sistemas, SOAP/WSDL |
| [[Stefanini - Java NET Angular]] | .NET, Angular, **Java, Spring Boot**, microservicios, microfrontends |
| [[Lider Tecnico Full Stack Angular NET]] | Angular, .NET, **Laravel/PHP**, liderazgo técnico, code review, AWS |
| Vacantes Java-first | Java 17, Spring Boot, Spring Data JPA, Spring Security, Maven, JUnit, Hibernate — **siempre bajo el encabezado de proyectos/formación** |

---

# Parte 5 — Qué agregar al terminar el laboratorio

Cada fase completada es una línea nueva y verdadera:

| Fase completada | Línea que habilita |
|---|---|
| Fase 1 | Pruebas unitarias y de integración con Testcontainers |
| Fase 2 | Docker multi-stage, despliegue en Azure App Service |
| Fase 3 | Gestión de secretos con Key Vault y Managed Identity |
| Fase 4 | Autenticación y autorización con OAuth2/OIDC y JWT |
| Fase 5 | Mensajería asíncrona, idempotencia, dead letter queue, Outbox |
| Fase 6 | Observabilidad: logs estructurados, métricas y trazabilidad distribuida |
| Fase 7 | Pipeline CI/CD con despliegue automatizado y rollback |

Siete de los huecos del CV, cerrados con evidencia propia en vez de con afirmaciones.

---

# Errores a evitar

- **Un solo CV para las 14.** El bloque de conocimientos y el título cambian por vacante; el resto no.
- **Listar tecnologías que no podés defender 10 minutos.** El entrevistador pregunta por lo que está escrito.
- **Mezclar proyectos propios con experiencia laboral** sin distinguirlos — es donde una adaptación legítima se vuelve una mentira.
- **Métricas inventadas.** Si no tenés el número, describí el alcance en vez de fabricar un porcentaje.
- **Poner Java entre las tecnologías de un empleo donde no lo usaste.** Con clientes bancarios (BCP vía Coforge) la verificación de antecedentes es estándar, y las consultoras te evalúan dos veces: ellas y el cliente. El costo real no es perder esa vacante — es que esa consultora deje de considerarte para las de .NET, donde sí competís con evidencia.
- **Omitir GitHub y LinkedIn.** Si el CV Java se apoya en proyectos propios, el link tiene que estar visible y el repo tiene que abrir.
