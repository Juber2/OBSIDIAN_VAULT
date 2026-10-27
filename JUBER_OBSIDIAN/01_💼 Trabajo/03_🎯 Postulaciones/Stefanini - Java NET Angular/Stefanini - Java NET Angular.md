---
tags:
  - Postulacion
  - Stefanini
  - Java
  - SpringBoot
  - dotNET
  - Angular
  - Microfrontends
Creado: 2026-08-30
Estado: Por postular
Empresa: Stefanini
Modalidad: No indicada
Sueldo: No indicado
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - JAVA
  - NET
  - ANGULAR
---

# Stefanini — Java + .NET + Angular

## Veredicto rápido

**Postular: sí, con prioridad alta.**

Es la vacante que mejor coincide hasta ahora con tu perfil objetivo porque combina:

- Angular en frontend.
- .NET/C# como backend principal e integración.
- Java/Spring Boot para microservicios.
- APIs REST, testing, seguridad, documentación e incidentes.

Las brechas principales son microfrontends, Oracle y MongoDB. El filtro determinante no es académico: debes poder acreditar tres años con .NET, un año con Java/Spring y experiencia Angular.

## Encaje estimado por escenario

| Experiencia comprobable | Encaje aproximado |
|---|---:|
| 3+ años .NET, 1+ Java/Spring y Angular profesional | 80-90 % |
| Cumples los años, pero no has trabajado con microfrontends/Oracle/Mongo | 70-80 % |
| Java/Spring o Angular son principalmente de estudio/proyectos | 55-70 % |
| No puedes acreditar tres años .NET | Riesgo alto por requisito explícito |

## Matriz de encaje

| Requisito | Encaje documental | Evidencia necesaria |
|---|---|---|
| .NET/C# backend | 🟢 Alto | servicios, integraciones, errores y pruebas |
| Java + Spring Boot | 🟢 Alto en contenido | experiencia o proyecto real de microservicio |
| APIs REST | 🟢 Alto | OpenAPI, contratos, versionado y resiliencia |
| Angular reciente | 🟢 Alto | feature moderna y decisiones de arquitectura |
| Microfrontends | 🟡 Parcial | host/remote, despliegue y shared dependencies |
| Microservicios | 🟢 Conceptual | límites, fallos, observabilidad y operación |
| Oracle | 🔴 No evidenciado | SQL Oracle e integración básica |
| MongoDB | 🔴 No evidenciado | modelado documental e índices |
| Testing | 🟢 Alto en contenido | unit, integration, regression y contract tests |
| Seguridad/cifrado | 🟡 Alto conceptual | ejemplos aplicados y gestión de llaves |
| Documentación | 🟡 | OpenAPI, ADR, diagramas y runbooks |
| Incidentes avanzados | 🟡 | historia real o simulación con RCA |

“Documental” describe lo visible en el vault; no reemplaza experiencia profesional.

## Requisitos excluyentes traducidos

### Java/Spring Boot 3 o 4

Debes poder:

- crear API REST con Spring MVC;
- usar DI, configuración, profiles y validación;
- manejar errores consistentemente;
- proteger endpoints con Spring Security;
- persistir con Spring Data/JPA;
- testear con JUnit, Mockito, integration tests y Testcontainers;
- exponer health, metrics y traces con Actuator/Micrometer;
- diseñar un servicio que falle de forma controlada.

Spring Boot 4 usa la generación actual de Spring Framework y requiere Java 17 como mínimo; confirma qué versión utiliza el cliente porque una aplicación Boot 3 existente no debe tratarse como si fuera Boot 4 sin evaluar migración. Ver [requisitos oficiales de Spring Boot](https://docs.spring.io/spring-boot/system-requirements.html).

Material: [[Spring Boot]], [[01 - Fundamentos Spring Boot]], [[02 - REST Controllers]], [[04 - Validaciones]], [[08 - Manejo de Errores]], [[09 - Swagger y OpenAPI]], [[Spring Security]], [[Testing en Java]].

### .NET/C# e integración

Debes poder:

- construir servicios ASP.NET Core;
- consumir Java y terceros con `HttpClientFactory`;
- definir timeout, retry y circuit breaker correctamente;
- propagar autenticación y correlation ID;
- transformar contratos sin acoplar dominios;
- manejar archivos de forma streaming;
- probar integraciones y compatibilidad.

Material: [[ASP.NET Core APIs]], [[02 - HttpClientFactory Refit y gRPC]], [[03 - Resiliencia con Polly]], [[05 - File Uploads y Streaming]], [[Integration Tests WebApplicationFactory]].

> [!note]
> La nota de uploads enlazada pertenece al bloque NestJS, pero los principios de streaming, límites y seguridad son transferibles. La implementación .NET debe hacerse con APIs de ASP.NET Core.

### Angular reciente

- Standalone components.
- TypeScript estricto.
- Routing/lazy loading.
- HttpClient, interceptors y estados de UI.
- Signals y RxJS con responsabilidades claras.
- Forms tipados.
- Auth y permisos.
- Testing, performance y accesibilidad.

Actualmente Angular mantiene varias versiones activamente soportadas; “versiones recientes” debe traducirse a la versión exacta del cliente y su estrategia de upgrade, no a perseguir cada novedad. Ver [compatibilidad oficial de Angular](https://angular.dev/reference/versions).

Material: [[ANGULAR]], [[Angular moderno versiones y estrategia de upgrade]], [[Feature Architecture Boundaries]], [[Testing Strategy Angular]].

## Microfrontends — brecha prioritaria

## Qué debes comprender antes de la herramienta

- Qué límite de negocio pertenece a cada frontend.
- Qué equipo posee y despliega cada parte.
- Qué responsabilidad tiene el shell/host.
- Cómo se cargan los remotes.
- Cómo se comparten Angular, RxJS y design system.
- Qué sucede cuando host y remote usan versiones incompatibles.
- Cómo se comparte identidad sin crear estado global caótico.
- Cómo se manejan navegación, permisos y comunicación.
- Cómo falla un remote sin derribar toda la aplicación.
- Cómo se prueban, despliegan y revierten independientemente.

Webpack Module Federation permite que builds separados expongan y consuman módulos en runtime, formando una aplicación unificada; los módulos compartidos y sus versiones requieren una estrategia explícita. Ver [Module Federation](https://webpack.js.org/concepts/module-federation/).

## Herramientas posibles

- Webpack Module Federation.
- Native Federation.
- Nx con configuración de host/remotes.
- single-spa u otra solución existente en el cliente.

No asumir la herramienta. Preguntar cuál utiliza Stefanini o el cliente.

Material del vault: [[Nx Monorepos Microfrontends]], [[Feature Architecture Boundaries]], [[ADR Decisiones Frontend Angular]].

## Laboratorio mínimo de microfrontends

1. Crear un shell Angular.
2. Crear remote `files`.
3. Cargar el remote por ruta.
4. Compartir Angular/RxJS evitando duplicados.
5. Implementar auth como contrato, no accediendo internamente al store del host.
6. Simular remote no disponible y mostrar fallback.
7. Desplegar host y remote por separado.
8. Probar compatibilidad y rollback.
9. Documentar por qué un monolito frontend también podría ser suficiente.

## Java ↔ .NET ↔ Angular

Modelo de integración que debes defender:

```text
Angular shell
  ├── remote de archivos ──► API .NET
  └── remote de tareas   ──► API .NET
                                  │
                                  └──► microservicio Java/Spring
                                           │
                                           ├── Oracle
                                           └── MongoDB si el modelo lo justifica
```

Contratos:

- OpenAPI para APIs síncronas.
- DTOs versionados y separados del dominio.
- Timeouts y cancelación.
- Retry solo para fallos transitorios y operaciones seguras/idempotentes.
- Circuit breaker para dependencia degradada.
- Correlation/trace ID de extremo a extremo.
- Auth propagada o credenciales service-to-service con mínimo privilegio.
- Contract tests para evitar que Java y .NET rompan integraciones.

## Oracle — mínimo necesario

No necesitas convertirte en DBA. Para entrevista y desarrollo:

- SQL relacional, joins, subqueries, CTE y funciones analíticas.
- Diferencias de dialecto y tipos.
- Sequences/identity.
- `DATE`, `TIMESTAMP`, `CLOB` y `BLOB`.
- Índices y `EXPLAIN PLAN`.
- Transacciones, locks y niveles de aislamiento.
- Connection pooling.
- Driver JDBC para Java y provider Oracle para .NET.
- Migrations y compatibilidad del esquema.
- PL/SQL a nivel de lectura si existe legacy.

El conocimiento de SQL Server/PostgreSQL es transferible, pero debes practicar sintaxis y herramientas Oracle.

## MongoDB — mínimo necesario

- Documento, colección y BSON.
- Diseño por patrones de acceso.
- Embedding vs references.
- Índices simples/compuestos y `explain`.
- Aggregation pipeline.
- Atomicidad por documento y cuándo usar transacciones.
- Schema validation.
- Paginación y crecimiento no controlado de documentos.
- Integración con Spring Data MongoDB o MongoDB Driver para .NET.

MongoDB no se agrega para almacenar logs ni porque “NoSQL escala más”. Debe resolver un modelo documental o patrón de acceso concreto.

## Testing funcional, unitario y de no impacto

“Pruebas de no impacto” suele significar demostrar que un cambio no rompe comportamiento existente:

- unit tests de reglas;
- integration tests por servicio;
- contract tests Java ↔ .NET ↔ frontend;
- regression suite;
- smoke tests después del despliegue;
- compatibilidad hacia atrás de API/eventos;
- pruebas de permisos y cifrado;
- performance baseline en flujos sensibles.

Preguntar qué significa exactamente el término dentro del cliente.

## Seguridad y cifrado

- TLS para datos en tránsito.
- Cifrado administrado para datos en reposo.
- Secret manager/vault para credenciales y llaves.
- Hash de contraseñas, no cifrado reversible.
- Autorización en backend.
- Rotación y auditoría de llaves.
- Cifrado de campos solo si el riesgo lo requiere.
- No implementar algoritmos criptográficos propios.
- Evitar datos sensibles en logs, errores y URLs.

Preparar una respuesta que distinga hashing, encryption y encoding.

## Observabilidad e incidentes

Spring Boot utiliza Micrometer Observation para métricas y trazas, mientras Actuator expone capacidades de salud y operación: [observabilidad de Spring Boot](https://docs.spring.io/spring-boot/reference/actuator/observability.html).

Debes poder:

- correlacionar Angular → .NET → Java → DB;
- revisar logs estructurados;
- analizar latencia, error rate y dependencias;
- usar health/readiness;
- identificar versión desplegada;
- mitigar, encontrar causa raíz y definir acción preventiva;
- escribir un postmortem sin culpar personas.

Material: [[08 - Distributed Tracing]], [[Logging tracing metrics OpenTelemetry]], [[Playbook de Incidentes dotNET]].

## Proyecto demostrable alineado

Crear un POC pequeño, no una plataforma completa:

- Angular shell.
- Remote `files` y remote `tasks`.
- API .NET para archivos/tareas.
- Servicio Spring Boot para una integración o auditoría.
- Un motor relacional como sistema principal.
- MongoDB solo para un agregado documental justificado.
- OpenAPI y contract tests.
- Auth, trazabilidad y pruebas.
- Docker Compose local.

El POC debe demostrar interoperabilidad, no simular una arquitectura empresarial innecesariamente grande.

## Preparación de siete días

| Día | Foco | Entregable |
|---|---|---|
| 1 | Evidencia y CV | historias .NET, Java y Angular con resultados |
| 2 | Spring Boot | API, JPA, seguridad, test y Actuator |
| 3 | .NET integración | cliente resiliente, archivos y contract test |
| 4 | Angular moderno | feature completa y decisiones Signals/RxJS |
| 5 | Microfrontends | shell, remote, fallback y deploy separado |
| 6 | Oracle + MongoDB | dos laboratorios pequeños y comparación |
| 7 | System design | demo, incident response y entrevista simulada |

Postular antes de completar el plan.

## Preguntas probables

### Java/Spring

- ¿Qué cambia conceptualmente entre Spring Boot 3 y 4?
- ¿Cómo separas responsabilidades de un microservicio?
- ¿Cómo manejas transacciones y errores?
- ¿Cómo instrumentas una API con Actuator/Micrometer?
- ¿Cómo proteges endpoints con Spring Security?

### .NET

- ¿Cómo integrarías un servicio Java que falla intermitentemente?
- ¿Cuándo aplicar retry y circuit breaker?
- ¿Cómo procesarías archivos grandes sin consumir toda la memoria?
- ¿Cómo versionas un contrato compartido?
- ¿Cómo pruebas la integración sin depender siempre del ambiente real?

### Angular/microfrontends

- ¿Por qué elegir microfrontends y cuándo no?
- ¿Qué pertenece al shell?
- ¿Cómo gestionas dependencias compartidas?
- ¿Cómo manejas auth y estado entre remotes?
- ¿Qué ocurre si un remote no carga?
- ¿Cómo despliegas y reviertes independientemente?

### Datos

- ¿Qué caso justificaría MongoDB frente a Oracle?
- ¿Embedding o referencias?
- ¿Cómo diagnosticas una consulta Oracle lenta?
- ¿Cómo manejas consistencia entre servicios y bases distintas?

### Producción

- Describe un incidente avanzado y su causa raíz.
- ¿Cómo correlacionas una petición entre tres tecnologías?
- ¿Qué pruebas ejecutarías para demostrar no impacto?
- ¿Cómo documentas una decisión de integración?

## Palabras clave para el CV

`C#`, `.NET`, `ASP.NET Core`, `Java`, `Spring Boot`, `microservices`, `REST APIs`, `Angular`, `TypeScript`, `microfrontends`, `OpenAPI`, `system integration`, `unit testing`, `integration testing`, `contract testing`, `security`, `encryption`, `incident resolution`, `technical documentation`.

Añadir `Oracle`, `MongoDB` o `Module Federation` solo si puedes respaldarlo con experiencia o laboratorio, especificando cuál.

## Preguntas para reclutamiento

1. ¿Qué porcentaje del trabajo corresponde a .NET, Java y Angular?
2. ¿Qué versión exacta de Spring Boot, Java y Angular utilizan?
3. ¿Qué solución de microfrontends usan?
4. ¿Oracle y MongoDB son ambos obligatorios o dependen del módulo?
5. ¿Cuál es la modalidad, sede y horario?
6. ¿Cuál es el rango salarial y tipo de contratación?
7. ¿Existe on-call o soporte fuera de horario?
8. ¿El cliente pertenece al sector seguros/financiero?
9. ¿Cuánto código es legacy y cuánto desarrollo nuevo?

## Decisión final

Esta debe ser una de tus postulaciones principales. El stack coincide mucho mejor con tu ruta que TCS o Cuidafarma. La preparación debe centrarse en **demostrar profundidad y conexión entre tecnologías**, no en añadir más herramientas al CV.

