---
tags:
  - Postulacion
  - Backend
  - Java
  - SpringBoot
  - dotNET
  - DDD
  - ArquitecturaHexagonal
  - EventDriven
  - DB2
Creado: 2026-08-30
Estado: Por evaluar
Empresa: No indicada
Modalidad: Híbrido
Ubicacion: San Isidro, Lima
Sueldo: S/6,000-S/8,000
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - JAVA
  - NET
---

# Backend Java + .NET — DDD, Hexagonal y DB2

## Veredicto rápido

**Postular si puedes acreditar una trayectoria senior combinando Java y .NET.**

Es una vacante backend enterprise, no Full Stack. El filtro principal son los **cinco años desarrollando con Java y .NET**; las mayores especializaciones son Spring Boot, microservicios, arquitectura moderna y tres años con DB2/SQL.

El rango de S/6,000-S/8,000 es superior al de varias vacantes anteriores, pero sigue exigiendo validar si los cinco años deben incluir ambos lenguajes simultáneamente y si DB2 es excluyente.

## Encaje estimado

| Escenario | Encaje aproximado |
|---|---:|
| 5+ años backend combinando Java/.NET y 3+ años DB2 | 85-95 % |
| 5+ años backend, ambos stacks sólidos, pero SQL Server/PostgreSQL en vez de DB2 | 68-82 % |
| Profundidad en uno de los dos stacks y experiencia menor en el otro | 55-70 % |
| Java o .NET principalmente académico | Riesgo alto |
| Menos de 5 años totales de backend | Riesgo alto por filtro explícito |

## Requisitos clasificados

### Excluyentes o altamente probables

- Formación técnica/universitaria afín.
- Cinco años como desarrollador backend Java y .NET.
- Java y Spring Boot.
- Microservicios.
- Arquitectura hexagonal, DDD y Event Driven aplicados.
- Tres años en DB2/SQL.

### Deseables

- Seguros, banca o sectores relacionados.
- Metodologías ágiles.
- Cloud.
- DevOps.

## Pregunta documental crítica

La frase “cinco años como desarrollador Java y .NET” es ambigua. Puede significar:

1. cinco años totales usando ambos stacks;
2. cinco años en Java y además cinco en .NET;
3. cinco años de backend y dominio actual de ambos.

No asumir la interpretación más estricta. Consultarla antes de descartar la postulación y adaptar el CV mostrando la línea temporal exacta de cada stack.

## Java y Spring Boot

Preparar evidencia de:

- APIs REST con Spring MVC;
- DI, configuración y profiles;
- validaciones y manejo consistente de errores;
- Spring Data/JPA y transacciones;
- seguridad y autorización;
- clientes HTTP resilientes;
- JUnit, Mockito, integration tests y Testcontainers;
- Actuator, health checks, métricas y trazas;
- diagnóstico de memoria, threads, GC y consultas lentas.

Material: [[JAVA]], [[Spring Boot]], [[Spring Data]], [[Spring Security]], [[Testing en Java]], [[Performance Java]].

## .NET backend

Preparar evidencia equivalente en:

- C# y ASP.NET Core Web APIs;
- DI, configuración y middleware;
- EF Core y transacciones;
- Problem Details, validación y contratos;
- OAuth/OIDC/JWT y autorización por policies;
- `HttpClientFactory`, timeout, retry y circuit breaker;
- pruebas unitarias y de integración;
- health checks, logs, métricas y trazas.

Material: [[ASP.NET Core APIs]], [[02 - HttpClientFactory Refit y gRPC]], [[03 - Resiliencia con Polly]], [[Integration Tests WebApplicationFactory]].

## Arquitectura hexagonal

Debes poder explicar y mostrar:

- dominio y casos de uso independientes de frameworks;
- puertos de entrada y salida;
- adapters HTTP, persistencia y mensajería;
- inversión de dependencias;
- pruebas del núcleo sin infraestructura;
- límites prácticos para evitar capas ceremoniales.

Ejemplo:

```text
REST / evento
     │
adapter de entrada
     │
puerto de aplicación
     │
caso de uso + dominio
     │
puertos de salida
  ┌──┴─────────┐
DB2 adapter   broker adapter
```

Material: [[03 - Arquitectura Hexagonal]], [[01 - Capas y Responsabilidades]], [[08 - ADR y Tradeoffs]].

## DDD aplicado

No basta definir entidades y value objects. Preparar:

- lenguaje ubicuo;
- bounded contexts;
- agregados y límites transaccionales;
- invariantes;
- entidades y value objects;
- repositorios como abstracción del dominio;
- domain services cuando una regla no pertenece a una entidad;
- domain events;
- anticorruption layer para sistemas legacy;
- relación entre bounded contexts y microservicios.

Evitar afirmar que cada entidad es un microservicio o que DDD exige arquitectura distribuida.

Material: [[04 - DDD Tactico]], [[05 - DTOs Mappers y Contratos]], [[07 - Modular Monolith]].

## Event Driven

Debes poder resolver:

- evento de dominio frente a evento de integración;
- publicación confiable mediante Outbox;
- consumidores idempotentes;
- entrega duplicada y mensajes fuera de orden;
- retry con backoff y DLQ;
- versionado de schemas;
- trazabilidad entre API, evento y consumidor;
- consistencia eventual y compensaciones;
- cuándo una llamada síncrona es más simple y correcta.

Material: [[Mensajeria]], [[07 - Comunicacion Asincrona - Kafka]], [[Mensajeria RabbitMQ Azure Service Bus Kafka]].

## Microservicios

Preguntas que debes poder defender:

- ¿Cómo determinas el límite de un servicio?
- ¿Cuándo conservar un monolito modular?
- ¿Cómo evitas una base compartida?
- ¿Cómo manejas consistencia entre servicios?
- ¿Cómo versionas APIs y eventos?
- ¿Cómo observas una transacción distribuida?
- ¿Cómo despliegas y reviertes de forma independiente?
- ¿Qué fallos introduce una arquitectura distribuida?

Material: [[Microservicios]], [[01 - Fundamentos Microservicios]], [[05 - Circuit Breaker - Resilience4j]], [[08 - Distributed Tracing]].

## DB2 y SQL

DB2 puede ser el mayor filtro específico. El conocimiento relacional es transferible, pero debes practicar el producto.

### Mínimo técnico

- tipos de datos y dialecto SQL de DB2;
- schemas, tablas, constraints, views y sequences/identity;
- joins, CTE, subqueries y funciones de ventana;
- índices y acceso a datos;
- planes de ejecución con `EXPLAIN`;
- estadísticas y optimización;
- transacciones, aislamiento, locks y deadlocks;
- paginación y procesamiento por lotes;
- stored procedures si el sistema es legacy;
- drivers JDBC y provider .NET;
- connection pooling, timeout y observabilidad;
- migrations y compatibilidad de cambios.

### Evidencia transferible

Si no tienes DB2 profesional, presentar experiencia con SQL Server, PostgreSQL u Oracle y demostrar un laboratorio DB2. No declarar tres años de DB2 si no existen.

## Banca y seguros

Es deseable, no obligatorio. Preparar vocabulario y problemas comunes:

- auditoría y trazabilidad;
- autorización y segregación de funciones;
- datos sensibles y cifrado;
- alta disponibilidad y recuperación;
- integridad, conciliación e idempotencia;
- procesamiento batch;
- integraciones legacy;
- ventanas de cambio y gestión de incidentes;
- cumplimiento y evidencia de controles.

La experiencia sectorial debe expresarse como contexto real, no como conocimiento memorizado.

## Cloud y DevOps

Al no especificar proveedor, preparar conceptos transferibles:

- contenedores y configuración por ambiente;
- secretos y mínimo privilegio;
- balanceo, autoscaling y health checks;
- base administrada y backups;
- logs, métricas, trazas y alertas;
- pipeline de build, tests, security scan y deploy;
- artefactos inmutables;
- migraciones controladas;
- rollback y feature flags;
- infraestructura como código a nivel básico.

No añadir AWS, Azure o GCP al CV por esta vacante salvo que tengas evidencia propia.

## Proyecto demostrable

Crear o adaptar un sistema de pagos/pólizas:

- servicio Java/Spring para el dominio principal;
- servicio .NET para integración o consulta;
- arquitectura hexagonal en ambos;
- bounded contexts documentados;
- persistencia relacional, idealmente laboratorio DB2;
- evento de integración con Outbox;
- consumidor idempotente;
- OpenAPI y schema de eventos;
- tests unitarios, integración y contrato;
- observabilidad distribuida;
- Docker y pipeline básico;
- ADR explicando por qué existen dos servicios y dos stacks.

## Preparación de siete días

| Día | Foco | Entregable |
|---|---|---|
| 1 | CV y años | línea temporal Java/.NET/DB2 y evidencia sectorial |
| 2 | Spring Boot | API, datos, seguridad, pruebas y observabilidad |
| 3 | .NET | API e integración equivalente |
| 4 | Hexagonal + DDD | diseño, código y ADR |
| 5 | Event Driven | Outbox, idempotencia, retry y DLQ |
| 6 | DB2 | laboratorio SQL, índices, explain y transacciones |
| 7 | System design | simulación de arquitectura bancaria/seguros |

Postular antes de completar la preparación.

## Preguntas probables

- ¿Cómo aplicaste arquitectura hexagonal en un proyecto real?
- ¿Qué pertenece al dominio y qué al adapter?
- ¿Cómo defines un agregado?
- ¿Cuándo usar un evento de dominio o integración?
- ¿Cómo evitas perder un evento después de confirmar una transacción?
- ¿Cómo haces idempotente un consumidor?
- ¿Cómo integras un servicio Java con otro .NET?
- ¿Cómo manejas contratos y errores entre stacks?
- ¿Cómo diagnosticas una consulta DB2 lenta?
- ¿Cómo investigas un deadlock?
- ¿Cuándo no usarías microservicios?
- Describe una decisión técnica y sus trade-offs.

## Palabras clave para el CV

`Java`, `Spring Boot`, `.NET`, `C#`, `ASP.NET Core`, `backend`, `microservices`, `hexagonal architecture`, `DDD`, `event-driven architecture`, `domain events`, `Outbox`, `idempotency`, `DB2`, `SQL`, `REST APIs`, `integration testing`, `observability`, `DevOps`, `Agile`.

Incluir DB2, banca o seguros únicamente si existe evidencia real.

## Preguntas para reclutamiento

1. ¿Los cinco años se exigen en cada stack o como experiencia backend total?
2. ¿Qué porcentaje del trabajo corresponde a Java y a .NET?
3. ¿Los tres años con DB2 son excluyentes?
4. ¿Qué versiones de Java, Spring Boot y .NET utilizan?
5. ¿Qué broker o plataforma de eventos utilizan?
6. ¿Los microservicios ya existen o se está modernizando un monolito?
7. ¿El proyecto pertenece a banca o seguros?
8. ¿Qué proveedor cloud utilizan?
9. ¿Existe on-call o soporte fuera de horario?
10. ¿Cuántos días presenciales requiere San Isidro?

## Decisión final

Es una postulación atractiva si tu historial puede demostrar ambos stacks y seniority. Refuerza una tendencia importante del radar: Java y .NET no aparecen como caminos excluyentes, sino como combinación frecuente en backend enterprise.

La preparación específica debe concentrarse en **DB2, arquitectura aplicada y evidencia de cinco años**, no en añadir otro framework.
