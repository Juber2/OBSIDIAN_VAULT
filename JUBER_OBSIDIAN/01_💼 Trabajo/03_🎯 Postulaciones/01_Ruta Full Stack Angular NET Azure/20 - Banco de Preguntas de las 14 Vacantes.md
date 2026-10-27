---
tags:
  - Postulaciones
  - Entrevistas
  - Preguntas
Creado: 2026-08-31
Relacionado:
  - 04 - Plan Detallado 3 Semanas (dia a dia)
  - 16 - Radar acumulativo de vacantes
  - 15 - Banco de preguntas Full Stack
---

# Banco de preguntas de las 14 vacantes

Consolidado de las secciones "Preguntas probables" que ya existen dentro de cada nota de vacante, más las **preguntas reales** que le hicieron a un postulante para el puesto de Coforge/BCP.

> [!important] La diferencia con [[15 - Banco de preguntas Full Stack]]
> Esa nota agrupa preguntas **por tecnología** (Angular, C#, SQL…) para estudiar un tema. Esta las ordena **por probabilidad real**, cruzando cuántas de las 14 vacantes hacen la misma pregunta. Si tenés poco tiempo, esta manda.

## Cómo usarla

**3 preguntas por día**, en el bloque 🌙 de la noche del [[04 - Plan Detallado 3 Semanas (dia a dia)]].

Para cada una: **responder en voz alta**, cronometrando 2 minutos, con esta estructura:

```
Definición breve  →  Ejemplo propio  →  Trade-off o cuándo NO
```

Después, autoevaluar:

| Marca | Significa |
|---|---|
| ✅ | La respondí completa, sin leer, con ejemplo propio |
| 🟡 | Sé el concepto pero me trabé o me faltó el ejemplo |
| ❌ | No pude — vuelve a la rotación en 3 días |

Una pregunta marcada 🟡 o ❌ **se repite**, no se descarta. Las ✅ se revisan una vez más antes de una entrevista.

---

# 🔴 NIVEL 1 — Preguntas que hacen 4 o más vacantes

Estas 12 son las de mayor retorno. Si solo preparás una parte del banco, que sea esta.

| # | Pregunta | La hacen | ✅🟡❌ |
|---|---|---|---|
| 1 | **¿Cómo evitás procesar dos veces el mismo mensaje / duplicar una operación?** | 5 vacantes | ☐ |
| 2 | **¿Cómo diagnosticás una consulta lenta?** | 5 vacantes | ☐ |
| 3 | **¿Cómo hacés rollback de un despliegue?** (y si incluye cambios de base) | 5 vacantes | ☐ |
| 4 | **¿Cuándo NO usarías microservicios?** | 4 vacantes | ☐ |
| 5 | **¿Cuándo aplicar retry y cuándo circuit breaker?** | 4 vacantes | ☐ |
| 6 | **¿Cómo versionás un contrato de API consumido por varios clientes?** | 4 vacantes | ☐ |
| 7 | **¿Cómo correlacionás una petición entre varios servicios?** | 4 vacantes | ☐ |
| 8 | **¿Cómo maneja una app secretos y credenciales sin guardarlas?** | 4 vacantes | ☐ |
| 9 | **¿Cómo publicás un evento junto con una transacción de base de datos?** (Outbox) | 3 vacantes | ☐ |
| 10 | **¿Qué causa un deadlock y cómo se reduce?** | 3 vacantes | ☐ |
| 11 | **¿Client credentials o authorization code? ¿Qué diferencia OAuth de autenticar un usuario?** | 3 vacantes | ☐ |
| 12 | **Describí una decisión técnica que tomaste y sus trade-offs.** | 4 vacantes | ☐ |

> [!tip] Por qué la #1 encabeza la lista
> "Evitar duplicados" aparece formulada distinto en cinco vacantes (idempotencia, reintentos, mensajes, transacciones), pero es **la misma respuesta**: at-least-once + operación idempotente + registro del ID procesado en la misma transacción. Prepararla una vez cubre cinco entrevistas.

---

# 🟠 NIVEL 2 — Confirmadas en entrevista real (Coforge/BCP)

Estas se las hicieron **de verdad** a un postulante para un puesto equivalente. No son estimación.

| # | Pregunta | ✅🟡❌ |
|---|---|---|
| 13 | ¿Qué es SAGA? ¿Coreografía u orquestación? ¿Qué pasa si falla el paso 3 de 5? | ☐ |
| 14 | ¿Qué es arquitectura hexagonal? ¿Qué es un puerto y qué un adaptador? | ☐ |
| 15 | ¿Cuál es la regla de dependencia en arquitectura en capas? ¿Qué es un modelo anémico? | ☐ |
| 16 | ¿Qué estados tiene un Circuit Breaker y qué dispara cada transición? | ☐ |
| 17 | ¿Qué diferencia hay entre Factory Method y Abstract Factory? | ☐ |
| 18 | ¿Por qué el Singleton con enum es el más seguro en Java? | ☐ |
| 19 | ¿Qué problema resuelve el patrón Repository frente a usar JPA/EF directo? | ☐ |
| 20 | ¿Cómo elimina Strategy un `if/else` gigante? Dame un caso propio. | ☐ |
| 21 | ¿Qué genera Lombok y por qué `@Data` es riesgoso en una entidad JPA? | ☐ |
| 22 | ¿Qué es Quarkus y cuándo lo elegirías sobre Spring Boot? | ☐ |
| 23 | ¿Cómo se produce un N+1 en JPA y cómo lo corregís? | ☐ |
| 24 | ¿Cómo funcionan las particiones y consumer groups en Kafka? | ☐ |
| 25 | ¿Qué es Cache-Aside? ¿Qué pasa si el caché y la base quedan inconsistentes? | ☐ |

---

# 🟡 NIVEL 3 — Por área

## Backend Java / Spring

| # | Pregunta | ✅🟡❌ |
|---|---|---|
| 26 | ¿Por qué inyección por constructor y no `@Autowired` en campo? | ☐ |
| 27 | ¿Qué diferencia hay entre `REQUIRED` y `REQUIRES_NEW` en `@Transactional`? | ☐ |
| 28 | ¿En qué casos `@Transactional` no se aplica aunque esté puesta? | ☐ |
| 29 | ¿Qué particularidades tiene trabajar con Java 11 frente a 17 o 21? | ☐ |
| 30 | ¿Cómo diagnosticás alto consumo de CPU o memoria en una JVM? | ☐ |
| 31 | ¿Cómo procesarías un archivo grande sin consumir toda la memoria? | ☐ |
| 32 | ¿Cómo probás un controller, un servicio y un repositorio? ¿Qué difiere en cada uno? | ☐ |
| 33 | ¿Cómo instrumentás una API con Actuator y Micrometer? | ☐ |

## Backend .NET

| # | Pregunta | ✅🟡❌ |
|---|---|---|
| 34 | ¿Qué ocurre internamente con `async`/`await` y por qué nunca usar `.Result`? | ☐ |
| 35 | ¿Por qué `DbContext` es scoped y no singleton? | ☐ |
| 36 | ¿Cuándo usar `AsNoTracking` y qué cuesta el tracking? | ☐ |
| 37 | ¿Cuándo elegirías Dapper sobre EF Core? (y por qué "es más rápido" no alcanza) | ☐ |
| 38 | ¿Cómo manejás errores globalmente con `ProblemDetails`? | ☐ |
| 39 | ¿Qué diferencia `401` de `403`? | ☐ |

## Datos

| # | Pregunta | ✅🟡❌ |
|---|---|---|
| 40 | ¿Qué índice crearías para esta consulta y cómo verificás que se usa? | ☐ |
| 41 | ¿Cómo interpretás un plan de ejecución? | ☐ |
| 42 | ¿Cómo hacés una migración sin detener el servicio (zero-downtime)? | ☐ |
| 43 | ¿`OFFSET` o keyset pagination para paginar millones de filas? | ☐ |
| 44 | ¿Qué diferencia hay entre ACID y BASE? ¿Cuándo aceptás consistencia eventual? | ☐ |
| 45 | ¿Qué caso justificaría MongoDB frente a una base relacional? ¿Embedding o referencias? | ☐ |
| 46 | ¿Qué diferencias relevantes hay entre SQL Server y PostgreSQL? | ☐ |
| 47 | ¿Cuándo usar un stored procedure en vez de lógica en la aplicación? | ☐ |

## Arquitectura y diseño

| # | Pregunta | ✅🟡❌ |
|---|---|---|
| 48 | ¿Cómo definís el límite de un microservicio? | ☐ |
| 49 | ¿Cómo definís un agregado en DDD? | ☐ |
| 50 | ¿Cuándo usar un evento de dominio y cuándo uno de integración? | ☐ |
| 51 | ¿Cómo implementarías CQRS sin sobreingeniería? | ☐ |
| 52 | ¿Cómo integrarías un servicio Java con uno .NET? ¿Cómo manejás contratos y errores entre stacks? | ☐ |
| 53 | ¿Cómo reducís latencia sin sacrificar consistencia? | ☐ |
| 54 | ¿Cómo aislarías datos en una aplicación multitenant? | ☐ |

## Seguridad

| # | Pregunta | ✅🟡❌ |
|---|---|---|
| 55 | ¿Cuál es la diferencia entre OAuth 2.0, OIDC y JWT? | ☐ |
| 56 | ¿Cómo protegés una API con Entra ID? | ☐ |
| 57 | ¿Por qué tener el scope correcto no alcanza para autorizar? (A01 OWASP) | ☐ |
| 58 | Nombrá 6 riesgos del OWASP Top 10 con un ejemplo concreto en código. | ☐ |
| 59 | ¿Cómo validás un webhook entrante? | ☐ |
| 60 | Ocultar un botón en el frontend no es seguridad — ¿cómo asegurás permisos de verdad? | ☐ |

## Cloud, contenedores y entrega

| # | Pregunta | ✅🟡❌ |
|---|---|---|
| 61 | ¿App Service, Container Apps o AKS? ¿Cuándo cada uno? | ☐ |
| 62 | ¿Cómo accede una app a Blob Storage sin guardar credenciales? | ☐ |
| 63 | ¿Qué diferencia hay entre liveness y readiness probes? | ☐ |
| 64 | ¿Cómo investigarías un `CrashLoopBackOff`? | ☐ |
| 65 | ¿Qué etapas tendría tu pipeline de CI/CD? | ☐ |
| 66 | ¿Qué problema resuelve Docker y cuál **no** resuelve? | ☐ |
| 67 | ¿Qué aporta un API Management delante de una API? | ☐ |
| 68 | El despliegue funcionó pero la API responde 500 — ¿dónde mirás? | ☐ |

## Integraciones y resiliencia

| # | Pregunta | ✅🟡❌ |
|---|---|---|
| 69 | ¿Cómo integrás un servicio externo que responde `429` y tiene límites diarios? | ☐ |
| 70 | ¿Cómo sabés si conviene reintentar después de un timeout? | ☐ |
| 71 | ¿Qué hacés con un mensaje venenoso (poison message)? | ☐ |
| 72 | ¿Cómo preservás el orden de mensajes sin destruir el throughput? | ☐ |
| 73 | ¿Qué pasa si el consumidor procesa y falla antes del ack? | ☐ |
| 74 | ¿Cuándo elegir Kafka frente a RabbitMQ o Service Bus? | ☐ |

## Conductuales y de rol

| # | Pregunta | ✅🟡❌ |
|---|---|---|
| 75 | Contame un error técnico que cometiste y qué aprendiste. | ☐ |
| 76 | Describí un incidente difícil: contexto, decisión, resultado. | ☐ |
| 77 | ¿Cómo decidís entre entregar rápido y reducir deuda técnica? | ☐ |
| 78 | ¿Cómo manejás un desacuerdo de arquitectura? | ☐ |
| 79 | ¿Qué buscás en un code review? | ☐ |
| 80 | ¿Cómo comunicás un retraso o incidente a alguien no técnico? | ☐ |
| 81 | ¿Qué harías en tus primeros 30 días? | ☐ |
| 82 | Describí tu experiencia full stack de los últimos cinco años. | ☐ |

---

# Rotación diaria — 3 preguntas por día

Alineada con los temas del [[04 - Plan Detallado 3 Semanas (dia a dia)]], para que la pregunta caiga el día que estudiaste el tema.

## Semana 1 — arquitectura y patrones

| Día | Preguntas | ☐ |
|---|---|---|
| Lun 31 ago | 14, 15, 48 | ☐ |
| Mar 1 sep | 13, 25, 50 | ☐ |
| Mié 2 sep | 16, 17, 18 | ☐ |
| Jue 3 sep | 19, 20, 49 | ☐ |
| Vie 4 sep | 75, 76, 81 | ☐ |
| Sáb 5 sep | *(proyecto — sin preguntas)* | — |
| Dom 6 sep | Repasar todas las 🟡 y ❌ de la semana | ☐ |

## Semana 2 — backend y datos

| Día | Preguntas | ☐ |
|---|---|---|
| Lun 7 sep | 21, 26, 33 | ☐ |
| Mar 8 sep | 23, 27, 28 | ☐ |
| Mié 9 sep | 34, 35, 36 | ☐ |
| Jue 10 sep | 2, 40, 44 | ☐ |
| Vie 11 sep | 10, 41, 46 | ☐ |
| Sáb 12 sep | *(proyecto)* | — |
| Dom 13 sep | Repasar 🟡 y ❌ | ☐ |

## Semana 3 — seguridad, mensajería y cloud

| Día | Preguntas | ☐ |
|---|---|---|
| Lun 14 sep | 55, 56, 57 | ☐ |
| Mar 15 sep | 1, 9, 24 | ☐ |
| Mié 16 sep | 4, 7, 53 | ☐ |
| Jue 17 sep | 3, 61, 63 | ☐ |
| Vie 18 sep | 5, 12, 77 | ☐ |
| Sáb 19 sep | *(proyecto)* | — |
| Dom 20 sep | Repasar todas las ❌ acumuladas de las 3 semanas | ☐ |

**Cubierto en la rotación: 45 de 82.** Las restantes son específicas de una vacante concreta (Oracle, SOAP, React, Android, VB6) — se preparan **solo si esa candidatura avanza**, revisando su nota individual.

---

## Antes de una entrevista concreta

No repasar las 82. Hacer esto:

1. Abrir la nota de **esa vacante** y leer su sección "Preguntas probables" propia.
2. Repasar las **12 del Nivel 1** — casi seguro cae al menos una.
3. Repasar las marcadas ❌ del área que esa vacante prioriza.
4. Tener listas **dos historias conductuales** (un error, un incidente) — caen en casi todas.
5. Preparar **qué vas a preguntar vos** al final: cada nota de vacante tiene una sección "Preguntas para reclutamiento" ya escrita.

## Señal de que estás listo

No es tener las 82 en ✅. Es poder:

- Responder cualquiera del **Nivel 1** sin preparación previa.
- Dar un **ejemplo propio** en al menos 3 de cada 4 respuestas — sin ejemplo, suena a teoría leída.
- Decir **"no sé, pero así lo investigaría"** en las que no sabés, sin inventar. Es mejor recibido que una respuesta vaga y falsa.
