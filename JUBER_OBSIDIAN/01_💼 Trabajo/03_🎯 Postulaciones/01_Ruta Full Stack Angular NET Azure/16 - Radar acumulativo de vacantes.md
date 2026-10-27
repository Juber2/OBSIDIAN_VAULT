---
tags:
  - Postulaciones
  - Radar
  - Requisitos
Creado: 2026-08-30
Actualizado: 2026-08-30
VacantesAnalizadas: 13
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Radar acumulativo de vacantes

Este radar decide qué agregar a la ruta a partir de demanda observada. Una tecnología mencionada una sola vez se registra, pero no desplaza automáticamente las prioridades principales.

## Vacantes incorporadas

| # | Empresa | Perfil real | Encaje | Decisión | Análisis |
|---|---|---|---:|---|---|
| 1 | TCS | Android + .NET + Azure | 65-75 % | Postular y aclarar Android | [[TCS - NET Azure Android]] |
| 2 | Cuidafarma | Backend Node.js + AWS serverless | 40-65 % según experiencia | Postulación stretch | [[Cuidafarma - Backend Node AWS]] |
| 3 | Stefanini | Java + .NET + Angular | 55-90 % según experiencia | Prioridad alta | [[Stefanini - Java NET Angular]] |
| 4 | Synopsis | Backend .NET/Java financiero | 45-90 % según dominio | Evaluar filtro financiero y contrato | [[Synopsis - Backend Senior Financiero]] |
| 5 | Empresa no indicada | Líder técnico Full Stack políglota | 50-85 % según liderazgo y stack | Postular y aclarar alcance | [[Lider Tecnico Full Stack Angular NET]] |
| 6 | ISP Consulting | Java/Spring + Angular + GCP/Kubernetes | 40-90 % según experiencia | Evaluar seniority y salario | [[ISP Consulting - Java Angular GCP]] |
| 7 | Empresa no indicada | Backend Java + .NET enterprise | 55-95 % según años y DB2 | Evaluar filtros de experiencia | [[Backend Java NET DDD DB2]] |
| 8 | TSOFT | Full Stack .NET + Angular + Oracle | 65-95 % según Oracle | Prioridad alta | [[TSOFT - NET Angular Oracle]] |
| 9 | Entelgy | Backend .NET Core + Azure | 65-95 % según Azure | Prioridad máxima | [[Entelgy - Backend NET Azure]] |
| 10 | Empresa no indicada | Full Stack Java + React | 55-95 % según años e inglés | Evaluar filtros y postular | [[Full Stack Java React San Isidro]] |
| 11 | Empresa no indicada | Full Stack Senior Java + Angular + AWS | 55-95 % según años y móvil | Postular y aclarar tecnología móvil | [[Full Stack Senior Java Angular AWS]] |
| 12 | Empresa no indicada | Java Senior enterprise + BIAN | 55-95 % según BIAN/banca | Evaluar especialización y postular | [[Java Senior BIAN OpenShift Azure APIM]] |
| 13 | Periferia IT Group | Migración VB6 a .NET 10 WinForms | 40-95 % según VB6 | Evaluar filtro legacy y contrato | [[Periferia - Migracion VB6 NET 10 WinForms]] |
| 14 | Synopsis | Senior FullStack — Java + Angular + Python + IA | 45-70 % según Java y Python | Evaluar; el diferenciador de IA sí lo tenés | *(rol nuevo, distinto al de [[Synopsis - Backend Senior Financiero]])* |
| 15 | TCS Perú | **Software Architect** Full Stack | 40-60 % — brecha de **rol**, no de tecnología | Norte de carrera; entrar a TCS por la otra vacante | [[TCS - Software Architect Full Stack]] |

> [!warning] La vacante 15 es de otra categoría
> Pide **"más de 5 años en arquitectura de software"** — no 5 años programando. La brecha no es una tecnología que se aprende: es un nivel de rol y evidencia de liderazgo. Su checklist técnico, en cambio, coincide casi punto por punto con el roadmap de estudio: sirve como **validación del plan**, no como objetivo de preparación.

## Señales nuevas (31/08/2026)

| Señal | Apariciones | Decisión |
|---|---:|---|
| **Herramientas de IA en el desarrollo** | **3** (Synopsis, Líder Técnico, TCS Architect) | 🔺 **Promovido** por la regla de 3 apariciones. Es el diferenciador más barato: ya lo hacés a diario |
| Arquitectura Hexagonal (explícita) | 2 | Ya cubierto — semana 1 |
| Quarkus | 2 (Coforge, TCS) | Ya cubierto — 9 notas |
| OpenShift | 2 | Cubierto vía Kubernetes |
| Microfrontends | 2 (Stefanini, TCS) | Condicionada — laboratorio pequeño solo si avanza |
| Python | 2 (Synopsis, TCS) | Registrar. Con 3 apariciones se promueve |
| Sonar / Fortify | 2 | Agregar al pipeline del laboratorio: es 1 línea de YAML |
| ELK / Dynatrace / Kibana | 1 | No estudiar: los conceptos son los de App Insights |
| PKCE (explícito) | 1 | Ya cubierto en [[04 - Entra ID y Seguridad de APIs]] |

## Datos salariales confirmados (31/08/2026)

Primeros rangos concretos del mercado, útiles para fijar expectativa:

| Vacante | Rango mensual | Modalidad | Años que pide |
|---|---|---|---:|
| Cuidafarma (Grupo Breca) — Backend Node + AWS | **S/ 6,500 – 7,500** | Presencial | 3 |
| Synopsis — Senior FullStack Java/Angular/Python | S/ 4,000 – 7,000 | Híbrido | Senior |

Cuidafarma paga más y pide **menos años** (3, y tenés ~5). El filtro no es la experiencia: es **Node.js profesional + AWS**.

## Frecuencia de requisitos

| Capacidad | Apariciones | Prioridad actual | Decisión |
|---|---:|---|---|
| .NET/C# | 8 | P0 dominante | Prioridad central y evidencia profesional |
| Node.js/JavaScript | 3 | Rama secundaria confirmada | Confirmar si “JAVA SCRIPT” es JavaScript; profundizar según rol |
| Java/Spring Boot | 7 | P0 segunda vía | Consolidar como alternativa backend real |
| PHP | 1 | Condicionada | Aprovechar Laravel existente; confirmar framework y porcentaje |
| React | 2 | P1 confirmado | Crear laboratorio Full Stack Java + React |
| Angular | 5 | P0 dominante | Demostrar versión moderna y arquitectura |
| REST/JSON e integraciones | 10 | P0 dominante | Contratos, resiliencia, SaaS y OpenAPI |
| Azure | 3 | P0 confirmado | Proyecto desplegado con seguridad y observabilidad |
| AWS | 4 | P0/P1 fuerte | Laboratorio desplegado y operable; Azure sigue como eje inicial |
| GCP | 1 | Condicionada crítica | Vocabulario y laboratorio Pub/Sub/GKE para ISP |
| Cloud serverless | 3 | P0 conceptual | Dominar funciones, eventos, límites y operación |
| App Service | 2 | P0 confirmado | Laboratorio obligatorio con slots y managed identity |
| Functions/Lambda | 3 | P0 confirmado | Implementar función idempotente y observable |
| Object Storage | 2 | P0 confirmado | Blob/S3 y acceso seguro |
| API Gateway/APIM | 3 | P0 confirmado | Preparar auth, policies, throttling y logs |
| CI/CD | 7 | P0 dominante | Practicar pipeline YAML, deploy y rollback |
| Git/Azure Repos/GitHub | 6 | P0 dominante | PRs, policies y trabajo colaborativo |
| Linux | 1 | P0 | Reforzar diagnóstico operativo |
| Agile/Scrum/Kanban/XP | 7 | P0 blando | Diferencias y ejemplos reales |
| Jira/Confluence | 1 | P0 blando | Manejo de usuario |
| Desarrollo móvil | 2 menciones: 1 Android | Condicionada | Confirmar tecnología; PWA/Ionic/nativo cambian la ruta |
| Observabilidad/RCA/incidentes | 4 | P0 dominante | App Insights, trazabilidad y soporte avanzado |
| PostgreSQL | 2 | P1 confirmado | Laboratorio desde .NET y comparación con SQL Server |
| SQL Server | 4 explícitas | P0 dominante | EF Core/Dapper, consultas, índices y alto volumen |
| EF Core/Dapper | 1 explícita | P0 específica .NET | Laboratorio comparativo de escritura y lectura |
| Bases relacionales | 11 | P0 dominante | SQL, procedures, índices, alto volumen y transacciones |
| NoSQL | 4 | P1 confirmado | Aprender selección y modelado, no solo producto |
| Oracle | 4 | P0/P1 fuerte | Laboratorios Java/.NET + Oracle con SQL y PL/SQL |
| PL/SQL | 2 explícitas | P1 confirmado | Procedures, functions, packages, triggers y tuning |
| MongoDB | 1 explícita | Condicionada | Modelado, índices y agregaciones |
| Testing unit/integration | 6 | P0 dominante | Añadir contract, regression, Postman/Newman y automatización |
| Seguridad/OAuth/JWT/cifrado | 7 | P0 dominante | OAuth, Entra ID, secretos, SSDLC y mínimo privilegio |
| Mensajería/eventos/DLQ | 4 | P0 dominante | Laboratorio con Outbox, idempotencia, retry, DLQ y observabilidad |
| Kafka | 1 explícita | Condicionada | Implementar si ISP avanza; comprender particiones y offsets |
| RabbitMQ | 1 explícita | Condicionada | Implementar si ISP avanza; exchanges, ack y DLX |
| Google Pub/Sub | 1 explícita | Condicionada crítica | Laboratorio pequeño por ser núcleo cloud de ISP |
| Microservicios | 7 explícitas | P0 dominante | Profundizar límites, datos y operación sin sobreingeniería |
| Microfrontends | 1 explícita | Condicionada importante | Laboratorio host/remote |
| Documentación técnica/OpenAPI | 4 | P0 confirmado | OpenAPI, ADR, diagramas y runbooks |
| SOAP/WSDL | 1 | Condicionada importante | Laboratorio para Synopsis |
| Servidores web/app | 1 | P1 para enterprise | IIS, Tomcat/JBoss y Nginx a nivel operativo |
| Sector financiero/seguros | 6 menciones: 1 excluyente | Filtro/contexto | Documentar evidencia; no se aprende en días |
| JBoss Fuse/Bantotal | 1 deseable | P2 específica | Contexto ahora; profundidad si avanza |
| Clean/Hexagonal/DDD/CQRS | 3 explícitas | P1 fuerte | Preparar caso aplicado, límites y trade-offs |
| DB2 | 1 con 3 años | Filtro específico | Laboratorio y evidencia transferible; confirmar si excluyente |
| DevOps genérico | 3 menciones | P1 transversal | Conectar CI/CD, contenedores, observabilidad e IaC |
| Docker | 1 explícita | P0 transversal | Mantener laboratorio multi-stage, Compose y health checks |
| Kubernetes/OpenShift | 2 menciones | P1 confirmado | Laboratorio de despliegue, seguridad y diagnóstico |
| Postman/Newman | 3 explícitas | P1 herramienta | Collections, scripts, CLI y ejecución en CI |
| Liderazgo técnico | 2 explícitas | P1 de evidencia | Preparar decisiones, mentoría, riesgos e incidentes |
| LLM/agentes/automatización | 1 deseable | Diferenciador P2 | Un POC seguro y evaluado; no nueva especialidad |
| IA asistida para desarrollo | 1 deseable | Herramienta P2 | Preparar un caso con revisión, tests y protección de datos |
| Alta concurrencia | 1 deseable | Condicionada | Performance end to end y laboratorio de carga |
| Entra ID/Managed Identity | 1 explícita | P1 específica Azure | Auth de usuario/servicio y acceso sin secretos |
| GitHub Actions | 1 explícita | P1 específica | Pipeline .NET con OIDC, tests, deploy y rollback |
| Inglés intermedio/avanzado | 1 explícita | Filtro crítico | Preparar entrevista, arquitectura e historias en inglés |
| Cloud sin proveedor | 2 deseables | P1 conceptual | Demostrar conceptos transferibles con Azure/AWS |
| Certificación específica de APIs | 1 deseable | P2 | No anteponerla a experiencia, CV y proyecto |
| XSLT/XML | 1 explícita | Condicionada | Laboratorio de transformación para integración legacy |
| SonarQube/Kiuwan/Fortify | 1 explícita | P1 específica | Quality gates, SAST y remediación en pipeline |
| BIAN | 1 explícita | Filtro específico | Preparar Service Landscape, Domains y contrato de ejemplo |
| Modernización legacy/VB6 | 1 explícita | Filtro específico | Characterization tests, inventario y migración incremental |
| WinForms/.NET desktop | 1 explícita | Condicionada | UI thread, separación de lógica, empaquetado y actualización |

## Cambios provocados por las vacantes

- Subir Azure Functions de “conocer” a “crear una integración mínima”.
- Practicar Azure Pipelines además de comprender CI/CD en general.
- Incluir API Management con una policy demostrable.
- Reforzar Linux orientado a soporte y diagnóstico.
- Añadir preparación mínima de Scrum, Jira y Confluence.
- Registrar Android como brecha condicionada, sin convertirlo aún en nueva especialidad.
- Confirmar como P0 los conceptos de serverless, API gateway, object storage y CI/CD, independientemente del proveedor.
- Crear una rama condicional Node.js + AWS para Cuidafarma.
- Reforzar PostgreSQL desde Node y no únicamente desde .NET/NestJS teórico.
- Añadir un laboratorio de mensajería con idempotencia, retry y DLQ.
- Mantener observabilidad como prioridad: ya aparece explícitamente en una vacante.
- AWS llega a dos apariciones: crear un laboratorio pequeño sin sustituir Azure como eje principal.
- Priorizar Stefanini por coincidencia directa con .NET + Angular y por aprovechar Java.
- Elevar integración entre sistemas a capacidad central: aparece en las seis vacantes.
- Crear laboratorio pequeño de microfrontends si Stefanini avanza.
- Estudiar Oracle y MongoDB como brechas específicas, sin desplazar SQL relacional general.
- Promover Oracle a laboratorio porque ya aparece en dos vacantes.
- Añadir SOAP/WSDL y OAuth client credentials como preparación específica de Synopsis.
- Mantener servidores de aplicaciones y JBoss Fuse como conocimiento enterprise condicional.
- Tratar experiencia financiera como filtro documental, no como checklist de estudio.
- Confirmar Angular + .NET como combinación repetida y coherente con el perfil objetivo.
- Promover PostgreSQL a laboratorio concreto porque aparece por segunda vez.
- Reconocer microservicios y CI/CD como demanda repetida, con énfasis en operación y rollback.
- Añadir evidencia de liderazgo técnico: decisiones, code review, mentoría, riesgos e incidentes.
- Registrar PHP y React como tecnologías de una sola vacante; no abrir rutas profundas hasta confirmar su peso.
- Mantener Node.js como rama secundaria: ya aparece dos veces, pero solo una vacante lo usa como núcleo.
- Tratar LLM/agentes como diferenciador opcional mediante una integración segura, observable y evaluada.
- Promover Java/Spring y Angular por alcanzar tres apariciones: Java queda confirmado como segunda vía profesional.
- Promover mensajería a laboratorio pequeño por su segunda aparición; practicar idempotencia, retry, DLQ y métricas antes que memorizar brokers.
- Registrar GCP, Pub/Sub y Kubernetes como críticos para ISP, pero todavía condicionados por una sola vacante.
- Elevar observabilidad y testing a demanda dominante por alcanzar tres y cuatro apariciones respectivamente.
- Mantener Kafka y RabbitMQ como implementaciones condicionadas: una sola vacante no obliga a dominar ambos en profundidad.
- Señalar el desajuste entre los años solicitados por ISP y su rango salarial antes de priorizar el proceso.
- Reforzar Java + .NET como combinación backend enterprise: ambos aparecen juntos en varias oportunidades.
- Promover Event Driven/mensajería por alcanzar tres apariciones; añadir Outbox y eventos de dominio/integración.
- Registrar arquitectura hexagonal y DDD como brecha específica, con un caso aplicado en vez de definiciones.
- Registrar DB2 como filtro de producto y preparar transferencia desde SQL Server/PostgreSQL/Oracle.
- Elevar el contexto banca/seguros: aparece en tres anuncios, aunque solo uno lo trata como filtro duro.
- Priorizar TSOFT por coincidencia directa con .NET + Angular y tres años de experiencia razonables.
- Promover Oracle por alcanzar tres apariciones; el laboratorio debe incluir .NET, SQL, índices y planes.
- Añadir PL/SQL como brecha específica: procedures, functions, packages y triggers con criterio.
- Separar IA asistida para programar de integración de LLM/agentes dentro de un producto.
- Preparar mantenimiento correctivo, regresión y despliegues con cambios de base de datos.
- Priorizar Entelgy por coincidencia casi completa con la vía backend .NET + Azure.
- Promover Azure por segunda aparición: integrar App Service, Functions, Key Vault, Service Bus y App Insights.
- Promover SQL Server por tercera aparición y añadir comparación práctica EF Core frente a Dapper.
- Elevar seguridad, CI/CD, Git, observabilidad, mensajería y microservicios por repetición acumulada.
- Añadir Entra ID, managed identity y GitHub Actions como implementaciones concretas del proyecto principal.
- Promover Clean Architecture/DDD/CQRS a laboratorio aplicado por segunda aparición arquitectónica explícita.
- Promover React por alcanzar dos apariciones; crear un laboratorio pequeño con Java/Spring.
- Elevar Java/Spring a cinco apariciones y mantenerlo como segunda vía laboral consolidada.
- Promover Postman por segunda aparición como herramienta práctica, sin confundirlo con diseño de APIs.
- Registrar inglés como filtro crítico para la vacante Java + React y preparar una entrevista técnica completa.
- Mantener cloud genérico a nivel transferible hasta confirmar el proveedor del proyecto.
- Elevar Java/Spring a seis apariciones y Angular a cinco: la combinación Java + Angular queda confirmada.
- Elevar AWS a cuatro apariciones y CI/CD a cinco; el laboratorio debe incluir seguridad, observabilidad y rollback.
- Registrar desarrollo móvil como requisito ambiguo repetido; confirmar PWA, híbrido o nativo antes de estudiar.
- Mantener certificaciones de APIs como P2: no desplazan cinco años de evidencia profesional.
- Reforzar selección SQL/NoSQL e integraciones externas resilientes para la vía Java + Angular.
- Elevar Java a siete apariciones, microservicios y seguridad a siete, y REST/integraciones a diez.
- Promover Kubernetes/OpenShift por segunda aparición; añadir despliegue, RBAC, routes y diagnóstico.
- Promover PL/SQL por segunda aparición y reforzar Oracle/SQL Server, ambos con cuatro.
- Añadir BIAN como filtro bancario específico, junto con un contrato de ejemplo.
- Registrar XSLT y el stack de calidad SonarQube/Kiuwan/Fortify como brechas de integración enterprise.
- Promover Postman/Newman por tercera aparición y ejecutarlo dentro del pipeline.
- Registrar migración VB6 → .NET 10/WinForms como rama especializada, sin desplazar el perfil web.
- Elevar Clean Architecture/DDD/SOLID por tercera aparición y aplicarlos con pragmatismo en legacy.
- Añadir characterization tests, convivencia, conciliación y rollback para modernizaciones.
- Mantener Periferia condicionada a experiencia real con VB6/desktop y aclarar duración del contrato.

## Umbrales para cambiar la ruta

- **1 aparición:** registrar y preparar vocabulario si la vacante interesa.
- **2 apariciones:** crear laboratorio pequeño.
- **3 o más apariciones:** promover a prioridad de estudio, si encaja con el perfil objetivo.
- **Requisito crítico de una oportunidad avanzada:** puede adelantarse aunque aparezca una vez.

## Próxima actualización

Al recibir otra vacante:

1. añadirla a la tabla;
2. incrementar frecuencias;
3. distinguir requisito técnico, herramienta de equipo y habilidad blanda;
4. actualizar prioridades solo con evidencia acumulada;
5. señalar contradicciones con Angular + .NET + Azure.
