---
tags:
  - Postulacion
  - Java
  - Senior
  - Microservicios
  - Oracle
  - OpenShift
  - Azure
  - BIAN
Creado: 2026-08-30
Estado: Por evaluar
Empresa: No indicada
Modalidad: No indicada
Ubicacion: No indicada
Sueldo: No indicado
Seniority: Senior
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - JAVA
---

# Java Senior — BIAN, OpenShift y Azure APIM

## Veredicto rápido

**Postular si puedes demostrar Java enterprise, APIs/microservicios y contexto bancario. Prioridad media-alta.**

El núcleo es Java legacy/enterprise, Oracle/SQL Server, microservicios, seguridad y gestión de APIs. Las brechas muy específicas son BIAN, XSLT, OpenShift, Kiuwan y Fortify.

El requisito mínimo explícito es dos años implementando microservicios/servicios/APIs. BIAN puede ser un filtro importante aunque no se indique cantidad de años.

## Encaje estimado

| Escenario | Encaje aproximado |
|---|---:|
| Java enterprise + APIs, banca, BIAN y OpenShift | 85-95 % |
| Java/APIs/Oracle sólidos; BIAN y OpenShift de laboratorio | 65-80 % |
| Java moderno sólido, pero poca experiencia legacy/bancaria | 55-70 % |
| Sin dos años implementando APIs/microservicios | Riesgo alto |

## Ambigüedades

- “JAVA SCRIPT” probablemente significa JavaScript, pero debe confirmarse.
- “Java SE 7+” puede indicar sistemas legacy que conviven con runtimes nuevos.
- No menciona Spring Boot ni otro framework Java.
- No aclara versión de OpenShift.
- No explica el nivel de experiencia esperado en BIAN.
- No indica modalidad, sede, contrato ni salario.

## Matriz de requisitos

| Capacidad | Importancia | Evidencia necesaria |
|---|---|---|
| Java SE 7+ | Crítica | compatibilidad, código enterprise y mantenimiento |
| APIs/microservicios | Crítica | 2+ años, despliegue, soporte y documentación |
| Oracle + SQL Server | Alta | SQL/PLSQL, procedures, tuning y transacciones |
| OpenShift | Alta | despliegue, configuración, seguridad y diagnóstico |
| Azure DevOps/APIM | Alta | pipeline, policies, seguridad y observabilidad |
| JWT/OAuth2 | Alta | flujos y validación aplicada |
| Swagger | Alta | contratos documentados y compatibles |
| XSLT | Específica | transformación XML y debugging |
| SonarQube/Kiuwan/Fortify | Alta | quality gates, SAST y remediación |
| Postman/Newman | Deseable | pruebas automatizadas en CI |
| BIAN | Crítica específica | service domains y contratos bancarios |
| Banca | Deseable fuerte | contexto e integraciones reales |

## Java SE 7+ y JavaScript

### Java

Preparar tanto mantenimiento como evolución:

- POO, colecciones, generics y excepciones;
- concurrencia y pools;
- JDBC y transacciones;
- compatibilidad Java 7/8/11/17+;
- diferencias de APIs y build;
- dependencias legacy;
- XML, JSON y servicios;
- profiling, memoria y threads;
- estrategia de migración incremental.

No usar features modernas en una solución que compile con Java 7. Preguntar la versión real de producción y la versión objetivo.

Material: [[JAVA]], [[Java Intermedio]], [[Java Avanzado]], [[Performance Java]].

### JavaScript

Si el requisito es realmente JavaScript:

- scope, closures y prototipos;
- módulos;
- promesas y async/await;
- event loop;
- manejo de errores;
- JSON/XML;
- consumo de APIs;
- seguridad y validación.

Confirmar dónde se utiliza: frontend, scripts de integración, policies o automatización.

## Microservicios y APIs

- diseño de recursos y contratos;
- versionado y compatibilidad;
- validación y errores;
- autenticación/autorización;
- timeout, retry, circuit breaker e idempotencia;
- logs, métricas y trazas;
- health/readiness;
- configuración y secretos;
- soporte y diagnóstico;
- rollback;
- documentación OpenAPI;
- pruebas de contrato.

Material: [[Microservicios]], [[01 - Fundamentos Microservicios]], [[05 - Circuit Breaker - Resilience4j]], [[08 - Distributed Tracing]], [[09 - Swagger y OpenAPI]].

## Oracle, SQL Server y PL/SQL

- SQL complejo, CTE y funciones de ventana;
- procedures/functions/packages;
- índices y planes de ejecución;
- binds y queries parametrizadas;
- transacciones, locks y deadlocks;
- connection pools;
- procesamiento batch;
- diferencias de dialecto Oracle/SQL Server;
- integración JDBC;
- scripts versionados y rollback.

Oracle alcanza cuatro apariciones en el radar y SQL Server también cuatro; ambos requieren práctica real.

## OpenShift

OpenShift se basa en Kubernetes, pero añade una plataforma y controles propios. Preparar:

- projects/namespaces;
- pods, deployments y services;
- routes;
- ConfigMaps y Secrets;
- readiness/liveness/startup probes;
- requests/limits;
- builds, images y registry;
- service accounts, RBAC y security context;
- logs, events y diagnóstico;
- rollout y rollback;
- diferencias prácticas frente a Kubernetes genérico.

Material: [[KUBERNETES]], [[01 - Fundamentos y Arquitectura]], [[02 - Pods Deployments y ReplicaSets]], [[03 - Services y Networking]], [[04 - ConfigMaps Secrets y Volumenes]], [[06 - Namespaces RBAC y Seguridad Basica]].

## Azure API Management

Debes poder diseñar policies para:

- validación de JWT;
- rate limiting y quotas;
- transformación de headers/body;
- routing/versionado;
- CORS;
- correlation ID;
- caching cuando sea seguro;
- ocultar secretos;
- logging y trazabilidad;
- products/subscriptions;
- developer portal y publicación.

APIM no reemplaza la autorización ni la validación del backend.

## Azure DevOps

- repositorios y pull requests;
- pipelines YAML;
- build/test/package;
- análisis Sonar/Kiuwan/Fortify;
- artefactos o imágenes;
- variables y secretos;
- despliegue por ambiente;
- approvals y gates;
- smoke tests;
- rollback y trazabilidad.

## XSLT

Preparar un laboratorio XML:

- XPath;
- templates y `match`;
- `apply-templates`;
- namespaces;
- variables y parámetros;
- condiciones y loops;
- transformación XML → XML/JSON/texto según herramienta;
- encoding;
- manejo de documentos grandes;
- pruebas con casos y schemas.

XSLT suele indicar integraciones legacy, SOAP, ESB o formatos bancarios. Confirmar dónde se ejecutan las transformaciones.

## Calidad y seguridad

### SonarQube

- bugs, vulnerabilities y code smells;
- duplicación y cobertura;
- quality profile/gate;
- deuda técnica;
- integración con PR/pipeline.

### Kiuwan/Fortify

- análisis estático/SAST;
- clasificación y priorización;
- falsos positivos;
- trazabilidad de remediación;
- no silenciar hallazgos sin justificación.

### Seguridad de APIs

- OAuth2 y OIDC;
- JWT: issuer, audience, firma y expiración;
- client credentials para servicio a servicio;
- scopes/roles;
- secretos y mínimo privilegio;
- TLS;
- validación y rate limiting;
- datos sensibles fuera de logs.

## Postman y Newman

- collections y environments;
- variables sin secretos;
- scripts pre-request;
- assertions positivas/negativas;
- autenticación;
- data-driven tests;
- ejecución Newman en CI;
- reportes y criterios de fallo.

## BIAN

BIAN es la brecha de dominio más importante. Preparar:

- propósito del estándar bancario;
- Service Landscape;
- Service Domains;
- Control Record;
- Behavior Qualifiers;
- operations/action terms;
- semantic APIs;
- alineación entre capacidad bancaria y microservicio;
- uso como referencia, no generación automática de arquitectura;
- adaptación al dominio y sistemas existentes.

Construir un ejemplo pequeño de dominio de pagos/cuentas con contrato OpenAPI inspirado en un Service Domain y documentar qué se adapta.

## Proyecto demostrable

- microservicio Java compatible con runtime confirmado;
- API OpenAPI;
- Oracle o SQL Server;
- JWT/OAuth2;
- policy equivalente de APIM;
- transformación XSLT;
- contenedor desplegado en OpenShift/Kubernetes local;
- pipeline Azure DevOps conceptual o ejecutable;
- SonarQube y Newman;
- README con mapeo BIAN y decisiones.

## Preparación de siete días

| Día | Foco | Entregable |
|---|---|---|
| 1 | CV/API | evidencia de dos años y contexto bancario |
| 2 | Java legacy | compatibilidad, JDBC y diagnóstico |
| 3 | Oracle/SQL Server | SQL, PL/SQL, índices y planes |
| 4 | OpenShift | deploy, probes, config y rollback |
| 5 | APIM/seguridad | policies JWT/OAuth, rate limit y logs |
| 6 | XSLT/calidad | transformación, Sonar y Newman |
| 7 | BIAN | diseño, demo y entrevista simulada |

Postular antes de completar el plan.

## Preguntas para reclutamiento

1. ¿“JAVA SCRIPT” significa JavaScript y dónde se utiliza?
2. ¿Qué versión exacta de Java y framework usan?
3. ¿Cuál es la versión y modalidad de OpenShift?
4. ¿BIAN es excluyente y qué Service Domains utilizan?
5. ¿Qué porcentaje corresponde a mantenimiento y desarrollo nuevo?
6. ¿Azure APIM es deseable o parte diaria del puesto?
7. ¿Cuál herramienta de calidad es obligatoria?
8. ¿Qué nivel de experiencia bancaria requieren?
9. ¿Cuál es el salario, contrato, modalidad y sede?
10. ¿Existe on-call o soporte fuera de horario?

## Decisión final

Es una oportunidad Java enterprise válida, pero más especializada que las anteriores. La prioridad depende de BIAN, banca y tolerancia a un stack legacy. La preparación diferencial es BIAN + OpenShift + XSLT + calidad automatizada; no otro framework.
