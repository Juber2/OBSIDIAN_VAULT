---
tags:
  - Postulacion
  - Synopsis
  - Backend
  - dotNET
  - Java
  - Financiero
  - Integraciones
Creado: 2026-08-30
Estado: Por evaluar
Empresa: Synopsis
Modalidad: Remota con asistencias puntuales
Ubicación: Lima
Contrato: Hasta 2026-12-31 con posible renovación
Horario: 08:30-18:30
Sueldo: Hasta S/ 6,000 en planilla
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - NET
  - JAVA
  - API
---

# Synopsis — Analista Backend Developer Senior financiero

## Veredicto rápido

**Postular si puedes sustentar experiencia financiera; de lo contrario, tratarla como postulación stretch.**

El núcleo técnico coincide bien con tu perfil:

- .NET/C# **o** Java, no necesariamente ambos.
- APIs REST e integración con proveedores SaaS.
- OAuth 2.0.
- SOAP y REST.
- Microservicios.
- SQL Server/Oracle/MySQL/MariaDB.
- Git/Azure Repos, testing y seguridad.

El principal filtro es **tres años en proyectos del sector financiero**. Esto no se resuelve estudiando. Debes demostrarlo mediante clientes, productos, responsabilidades y flujos reales.

## Condiciones a evaluar

| Condición | Lectura imparcial |
|---|---|
| Seniority | Senior, con dominio sectorial requerido |
| Contrato | Hasta 31/12/2026; renovación no garantizada |
| Salario | Hasta S/ 6,000; inferior a Cuidafarma y ajustado para “senior” |
| Horario | 8:30-18:30; confirmar refrigerio y horas efectivas |
| Modalidad | Remota con oficina puntual, aunque el anuncio también indica híbrido |
| Estabilidad | Debe ponderarse por duración contractual |

La oportunidad puede ser valiosa por experiencia bancaria, pero no debe evaluarse solo por el stack.

## Encaje estimado por escenario

| Situación real | Encaje aproximado |
|---|---:|
| 3+ años financieros y 3+ años backend .NET/Java | 80-90 % |
| Experiencia en pagos/beneficios cercana al sector, pero no tres años bancarios | 60-75 % |
| Buen backend sin experiencia financiera comprobable | 45-60 % |
| Sin grado universitario requerido | Riesgo de filtro formal |

## ¿Pluxee puede contar como sector financiero?

**Posiblemente, pero no automáticamente.** Puede presentarse como experiencia relacionada con pagos o beneficios si tus funciones reales incluyeron, por ejemplo:

- tarjetas, saldos o transacciones;
- integraciones con sistemas de pago;
- seguridad y datos sensibles;
- conciliaciones;
- alta disponibilidad o soporte de procesos críticos;
- trazabilidad y auditoría.

No afirmar “tres años en banca” si no corresponde. Una formulación honesta podría ser “experiencia en plataforma de beneficios/pagos con procesos transaccionales”, indicando duración y responsabilidades reales.

## Matriz de encaje

| Requisito | Encaje documental | Acción |
|---|---|---|
| .NET/C# o Java | 🟢 Alto | Elegir la tecnología con mayor experiencia |
| Buenas prácticas de APIs | 🟢 Alto | Preparar diseño y ejemplos propios |
| Integración SaaS REST | 🟡 Alto conceptual | Demostrar resiliencia, auth y rate limits |
| Microservicios | 🟢 Cubierto | Explicar límites y fallos, no solo definición |
| OAuth 2.0 | 🟢 Cubierto | Dominar flows aplicables a integraciones |
| SOAP/WSDL | 🟡 Parcial | Crear un cliente y manejar faults/XML |
| SQL Server | 🟢 Alto | Preparar alto volumen y tuning |
| Oracle | 🟡 Brecha repetida | Realizar laboratorio específico |
| MySQL/MariaDB | 🟡 Transferible | Comparar dialecto y operación básica |
| Azure Repos/Git | 🟢 Transferible | Practicar PRs y branch policies |
| Scrum/XP/Kanban | 🟡 | Preparar diferencias y ejemplos |
| Servidores de aplicaciones | 🟡 | Comprender roles y diagnóstico |
| JBoss Fuse | 🔴 No evidenciado | Aprender Camel/rutas/EIPs si avanza |
| Ciclo seguro | 🟢 Conceptual | Estructurar SSDLC y controles |
| Testing automatizado | 🟢 Alto en contenido | Mostrar integración/contratos |
| Bantotal | 🔴 No evidenciado | Deseable; aprender contexto, no implementación |
| 3 años financieros | ❓ Debe comprobarse | Documentar evidencia real |

## Elegir .NET o Java

La oferta dice “.NET C# **o** Java”. Para esta candidatura:

1. Elegir la tecnología con más años profesionales.
2. Colocarla primero en el título y resumen del CV.
3. Utilizar la otra como complemento de interoperabilidad.
4. No repartir la preparación 50/50 si una es claramente más fuerte.

Si tu experiencia comprobable principal es .NET, la ruta recomendada es ASP.NET Core + SQL Server + OAuth + SOAP + microservicios. Java/Spring queda como ventaja adicional.

## APIs e integraciones SaaS

Debes poder diseñar una integración completa, no solo hacer un `GET`:

- contrato, versionado y compatibilidad;
- autenticación OAuth 2.0/API key/mTLS según proveedor;
- secretos y rotación;
- timeout por operación;
- retry con backoff y jitter;
- `429` y `Retry-After`;
- circuit breaker;
- idempotency key;
- paginación y límites;
- webhooks y validación de firma;
- reconciliación cuando el estado queda incierto;
- logs y correlation ID;
- SLA, métricas y alertas;
- sandbox vs producción.

Material: [[Diseno de APIs empresariales]], [[02 - HttpClientFactory Refit y gRPC]], [[03 - Resiliencia con Polly]], [[API]].

## OAuth 2.0 para esta vacante

Priorizar:

### Client Credentials

Para integración servicio-a-servicio donde el cliente actúa en nombre propio. Debes explicar:

- client ID y secreto/certificado;
- token endpoint;
- scopes y audience;
- caché y renovación del token;
- manejo de `401/403`;
- almacenamiento y rotación segura;
- mínimo privilegio.

### Authorization Code + PKCE

Para autorización con usuario y aplicaciones públicas/web. Comprender redirect URI, state, PKCE, access/refresh tokens y protección del navegador.

OAuth 2.0 permite acceso delegado o acceso del cliente en nombre propio; el grant client credentials está destinado a clientes confidenciales. Ver [RFC 6749](https://www.rfc-editor.org/info/rfc6749/).

Material: [[05 - OAuth2 OpenID Connect]], [[03 - OAuth2 y OpenID Connect]], [[Preguntas Seguridad JWT]].

## SOAP — brecha prioritaria para Synopsis

### Conceptos

- SOAP envelope, header y body.
- XML namespaces.
- WSDL y XSD.
- Operations, messages, bindings y endpoints.
- SOAP Fault.
- Document/literal vs RPC a nivel conceptual.
- WS-Security si el proveedor lo utiliza.
- TLS, certificados y mTLS.

### En implementación

- Generar un proxy tipado desde WSDL.
- Configurar endpoint y binding por ambiente.
- Serializar/deserializar XML correctamente.
- Definir timeouts y límites de tamaño.
- Registrar metadata sin exponer contenido sensible.
- Convertir SOAP Fault a error de dominio.
- Simular proveedor con un stub para pruebas.
- Detectar cambios incompatibles de WSDL.

En .NET, WCF y sus clientes representan el modelo clásico de interoperabilidad SOAP; Microsoft documenta clientes generados para invocar operaciones descritas por el servicio: [WCF Client Overview](https://learn.microsoft.com/dotnet/framework/wcf/wcf-client-overview).

## Laboratorio SOAP mínimo

1. Conseguir o crear un WSDL de prueba.
2. Generar cliente .NET o Java.
3. Invocar una operación exitosa.
4. Manejar un SOAP Fault.
5. Añadir timeout y autenticación.
6. Mockear el servicio en integration tests.
7. Mapear XML externo a un modelo interno.
8. Documentar por qué no expondrías directamente los tipos generados al dominio.

## Microservicios financieros

Preparar criterios sobre:

- límites por capacidad de negocio;
- consistencia fuerte vs eventual;
- idempotencia de operaciones monetarias;
- outbox e inbox/deduplicación;
- audit trail inmutable;
- autorización y segregación de funciones;
- timeouts/retries/circuit breaker;
- trazabilidad distribuida;
- versionado de contratos;
- reconciliación;
- despliegue y rollback;
- evitar transacciones distribuidas innecesarias.

Material: [[Microservicios .NET]], [[Microservicios]], [[01 - Eventos Integracion y Outbox]], [[05 - Transacciones Unit of Work y Concurrencia]].

## Bases de datos de alto volumen

No necesitas dominar cuatro motores como DBA. Debes dominar principios y profundizar en uno o dos.

### Imprescindible

- modelado y constraints;
- índices según patrones de consulta;
- planes de ejecución;
- paginación keyset cuando `OFFSET` deja de escalar;
- transacciones y aislamiento;
- locks, deadlocks y concurrencia optimista;
- batching/bulk operations;
- connection pooling;
- particionamiento y archivado cuando el volumen lo justifica;
- migrations compatibles y zero-downtime;
- réplicas de lectura y límites de consistencia;
- medición antes de optimizar.

### Prioridad por motor

1. SQL Server: profundidad y evidencia.
2. Oracle: laboratorio, porque ya aparece en Stefanini y Synopsis.
3. MySQL/MariaDB: diferencias principales y práctica SQL transferible.

Material: [[SQL SERVER]], [[02 - Execution Plans]], [[02 - Query Store]], [[04 - Deadlocks]], [[02_📚 Estudio/15_🗄️ SQL Server/13_ Senior SQL Server/03 - Migraciones Zero-Downtime|Migraciones Zero-Downtime en SQL Server]], [[05 - SQL Server vs Postgres Comparativa Final]].

## Azure Repos y flujo colaborativo

- repositorio, branches y permisos;
- pull request, reviewers y comentarios;
- branch policies;
- build validation;
- enlaces con work items;
- estrategia de ramas;
- tags/releases;
- secret scanning y credenciales.

Tu experiencia Git es transferible. Practicar la interfaz y vocabulario de Azure Repos, no reaprender control de versiones.

## Scrum, XP y Kanban

| Enfoque | Debes saber explicar |
|---|---|
| Scrum | roles/accountabilities, eventos, backlog e incremento |
| Kanban | flujo continuo, WIP limits, lead/cycle time y cuellos |
| XP | TDD, pair programming, refactoring, CI, simple design y feedback |

Preparar ejemplos reales: cómo manejaste una urgencia, deuda técnica, revisión o defecto productivo.

## Servidores y reverse proxies

| Tecnología | Papel principal |
|---|---|
| IIS | hosting/reverse proxy típico del ecosistema Microsoft |
| Tomcat | servlet container para aplicaciones Java web |
| JBoss EAP | application server Jakarta EE empresarial |
| Nginx | servidor web y reverse proxy eficiente |
| Apache HTTP Server | servidor web/reverse proxy modular |

Debes entender TLS termination, virtual hosts, headers proxy, balanceo, health checks, logs, puertos, procesos y despliegue; no administrar todos a nivel experto.

Material: [[01 - Servidores Web Nginx Apache]], [[LINUX]], [[04 - Health Checks]].

## JBoss Fuse

Red Hat Fuse es una plataforma de integración basada en Apache Camel. Puede ejecutarse sobre Spring Boot, Apache Karaf o JBoss EAP y ofrece componentes para conectar protocolos/productos y modelar rutas de integración. Ver [documentación de Red Hat Fuse/Camel](https://docs.redhat.com/en/documentation/red_hat_fuse/7.13/html/apache_camel_component_reference/index).

Aprender si la candidatura avanza:

- route, endpoint, component y exchange;
- producer/consumer;
- Enterprise Integration Patterns;
- content-based router, splitter, aggregator y dead letter channel;
- transformaciones y validación;
- REST/SOAP/JMS/file integration;
- retries, error handlers y observabilidad;
- despliegue en el runtime usado por el cliente.

No necesitas instalar toda la plataforma antes de postular. Primero confirma versión y cuánto código Fuse se mantiene.

## Bantotal

Bantotal es un core bancario que cubre procesos de misión crítica como clientes, cuentas, préstamos, tarjetas y contabilidad en línea. [Descripción oficial de Bantotal Core](https://www.bantotal.com/core.html).

Como es deseable, prepara contexto:

- qué es un core bancario;
- transacciones monetarias y contabilidad;
- saldos, conciliación y auditoría;
- disponibilidad 24/7;
- integraciones con canales y proveedores;
- idempotencia, seguridad y trazabilidad.

No fingir conocimiento de APIs o módulos específicos de Bantotal sin experiencia.

## Ciclo de desarrollo seguro

- threat modeling ligero;
- requisitos de seguridad y clasificación de datos;
- revisión de diseño;
- SAST, dependency y secret scanning;
- pruebas unitarias/integration/security;
- code review;
- separación de ambientes y secretos;
- aprobación y despliegue trazable;
- monitoreo y respuesta;
- parcheo y gestión de vulnerabilidades.

Material: [[06 - OWASP para APIs NET]], [[03 - Seguridad Backend Java]], [[Seguridad GitHub]].

## Proyecto demostrable

Crear un laboratorio de integración financiera acotado:

```text
Cliente
  └── REST API .NET o Java
         ├── OAuth2 client credentials → SaaS mock
         ├── SOAP client → proveedor legacy mock
         ├── SQL Server → operaciones e idempotencia
         └── logs/traces/audit trail
```

Debe incluir:

- OpenAPI.
- Cliente SOAP generado.
- Retry/timeout/circuit breaker.
- Idempotency key.
- Índice y query plan.
- Unit/integration/contract tests.
- Pipeline.
- Runbook de incidente.

## Preparación de cinco días

| Día | Foco | Evidencia |
|---|---|---|
| 1 | CV y dominio financiero | tres historias verificables |
| 2 | OAuth + integración SaaS | client credentials y resiliencia |
| 3 | SOAP | cliente WSDL, fault y test |
| 4 | SQL de alto volumen | plan, índice, concurrencia y paginación |
| 5 | Microservicios + servidores + simulación | diseño y entrevista senior |

JBoss Fuse y Bantotal pasan a estudio intensivo solo si la candidatura avanza.

## Preguntas probables

### APIs e integración

- ¿Cómo integras un SaaS que responde 429 y tiene límites diarios?
- ¿Cómo sabes si debes reintentar después de un timeout?
- ¿Cómo evitas duplicar una transacción?
- ¿Cómo versionas un contrato usado por varios consumidores?
- ¿Cómo validas un webhook?

### OAuth/SOAP

- ¿Client credentials o authorization code?
- ¿Qué diferencia OAuth de autenticación de usuario?
- ¿Qué contiene un WSDL?
- ¿Cómo manejas SOAP Faults?
- ¿Cómo proteges secretos y certificados?

### Datos

- ¿Cómo optimizas una consulta con millones de filas?
- ¿OFFSET o keyset pagination?
- ¿Qué causa un deadlock?
- ¿Cómo haces una migración sin detener el servicio?
- ¿Cómo garantizas integridad e idempotencia?

### Arquitectura/operación

- Diseña una integración con core bancario.
- ¿Qué pondrías en el audit trail?
- ¿Cómo diagnosticas una discrepancia de saldo?
- ¿Qué papel cumple un bus de integración?
- ¿Cómo despliegas y haces rollback con bajo riesgo?

## Palabras clave para el CV

Elegir una vía principal:

`Senior Backend Developer`, `.NET`, `C#`, `ASP.NET Core`, `REST APIs`, `OAuth 2.0`, `SOAP`, `WSDL`, `microservices`, `SaaS integrations`, `SQL Server`, `high-volume data`, `Git`, `Azure Repos`, `unit testing`, `integration testing`, `secure SDLC`, `financial systems`.

Agregar Java, Oracle, JBoss Fuse o Bantotal únicamente con el nivel real correspondiente.

## Preguntas para reclutamiento

1. ¿Los tres años financieros son excluyentes y aceptan experiencia en pagos/beneficios?
2. ¿El puesto se cubre principalmente con .NET o Java?
3. ¿Qué porcentaje de integraciones es SOAP frente a REST?
4. ¿Cuál es el motor principal de base de datos?
5. ¿JBoss Fuse y Bantotal son obligatorios en el trabajo diario?
6. ¿Qué versiones de frameworks y servidores utilizan?
7. ¿Por qué el contrato termina el 31/12 y cómo se decide la renovación?
8. ¿La remuneración máxima de S/ 6,000 es negociable?
9. ¿Con qué frecuencia se asiste a oficina?
10. ¿Existe soporte on-call o fuera del horario indicado?

## Decisión final

Técnicamente es una buena candidatura para tu perfil backend, especialmente por .NET/Java e integraciones. Comercialmente es menos atractiva que Cuidafarma por salario y duración, y profesionalmente depende mucho de que tu experiencia pueda considerarse financiera. Postular tiene sentido si deseas entrar o profundizar en banca y puedes explicar esa relación con honestidad.
