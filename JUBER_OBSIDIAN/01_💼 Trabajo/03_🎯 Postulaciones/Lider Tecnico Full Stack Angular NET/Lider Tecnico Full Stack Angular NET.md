---
tags:
  - Postulacion
  - LiderTecnico
  - FullStack
  - Angular
  - dotNET
  - AWS
  - IA
Creado: 2026-08-30
UltimaRecepcion: 2026-08-30
VecesRecibida: 2
Estado: Por evaluar
Empresa: No indicada
Modalidad: Híbrido
Ubicacion: Miraflores, Lima
Sueldo: S/5,500-S/6,500
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - ANGULAR
  - LARAVEL
  - NESTJS
---

# Líder Técnico / Full Stack — Angular, .NET y stack políglota

## Veredicto rápido

**Postular: sí, pero confirmar el alcance antes de convertirla en prioridad máxima.**

La coincidencia principal es buena: Angular, TypeScript, .NET/C#, REST, SQL Server, PostgreSQL, Docker, microservicios y CI/CD encajan directamente con la ruta objetivo. AWS ya no es una aparición aislada en el radar y merece un laboratorio pequeño.

La advertencia es la amplitud: además solicita PHP, React, Node.js, liderazgo e IA. Eso puede significar un líder transversal sobre varios equipos, pero también mantenimiento de muchos sistemas con poca profundidad. El anuncio no indica años de experiencia, tamaño del equipo, responsabilidades reales de liderazgo ni cuáles tecnologías son centrales.

El rango de **S/5,500-S/6,500** parece ajustado para un liderazgo técnico que de verdad sea responsable de arquitectura, personas, entregas y operación sobre esta cantidad de stacks. No descarta la vacante, pero obliga a aclarar el alcance.

## Encaje estimado por escenario

| Escenario comprobable | Encaje aproximado |
|---|---:|
| Angular/.NET sólidos, liderazgo real y experiencia con AWS/Docker | 75-85 % |
| Angular/.NET sólidos; PHP, React, Node o AWS son secundarios | 65-78 % |
| Buen perfil técnico, pero sin historias de liderazgo | 50-65 % |
| La empresa exige profundidad profesional simultánea en todos los stacks | Encaje bajo y posible rol mal definido |

El porcentaje definitivo depende menos de haber tocado cada tecnología y más de qué stack ocupa la mayor parte del puesto.

## Lectura imparcial del anuncio

### Núcleo probable

- Angular y TypeScript.
- .NET Core/C#.
- APIs REST e integraciones end to end.
- SQL Server y PostgreSQL.
- Arquitectura de microservicios.
- Docker, CI/CD y AWS.
- Coordinación técnica bajo Scrum.

### Capacidades secundarias o de interoperabilidad

- PHP, posiblemente para sistemas existentes.
- React, probablemente para otro producto o equipo.
- Node.js, posiblemente para integraciones o servicios puntuales.

### Diferenciador, no requisito central

- Integración de modelos de lenguaje.
- Agentes y automatización de flujos.
- Certificaciones de IA.

No conviene interpretar “se valorará” como obligación previa. Un proyecto pequeño pero operable de integración con IA vale más como evidencia que memorizar teoría o acumular una certificación sin implementación.

## La brecha más importante: demostrar liderazgo

Un líder técnico no es únicamente quien conoce más frameworks. Debes preparar evidencia de que puedes:

- convertir requisitos ambiguos en una solución y un plan entregable;
- tomar decisiones arquitectónicas y explicar sus trade-offs;
- dividir trabajo, identificar dependencias y anticipar riesgos;
- revisar código sin convertirse en cuello de botella;
- acordar contratos entre frontend, backend y terceros;
- acompañar a otros desarrolladores y elevar el estándar del equipo;
- coordinar con producto, QA, infraestructura y seguridad;
- responder ante incidentes y comunicar estado, impacto y recuperación;
- proteger la mantenibilidad sin frenar la entrega;
- documentar decisiones mediante ADR, diagramas y runbooks.

Preparar al menos cuatro historias STAR:

1. una decisión técnica difícil;
2. un incidente o problema de producción;
3. una mejora de calidad o proceso que ayudó al equipo;
4. un desacuerdo técnico resuelto con evidencia.

Si no has tenido el cargo formal, utiliza ejemplos de liderazgo sin autoridad: diseño, mentoría, code review, coordinación o responsabilidad completa de una entrega. No inventar gestión de personas.

Material: [[02_📚 Estudio/10_🔺 Laravel/13_ Senior Laravel/05 - Liderazgo Tecnico Backend|Liderazgo técnico backend]], [[02_📚 Estudio/10_🔺 Laravel/13_ Senior Laravel/02 - Decisiones Arquitectonicas|Decisiones arquitectónicas]], [[01 - Code Review Senior Laravel]], [[ADR Decisiones Frontend Angular]].

## Priorización técnica para esta vacante

| Orden | Capacidad | Profundidad objetivo |
|---:|---|---|
| 1 | Liderazgo técnico | historias reales, decisiones, coordinación y riesgos |
| 2 | Angular + .NET | profundidad profesional y solución end to end |
| 3 | APIs e integración | contratos, auth, errores, resiliencia y trazabilidad |
| 4 | SQL Server + PostgreSQL | modelado, consultas, índices y transacciones |
| 5 | Docker + CI/CD | build y despliegue reproducible |
| 6 | AWS | arquitectura y laboratorio desplegado |
| 7 | PHP/Laravel | lectura, mantenimiento e integración si se confirma |
| 8 | React y Node.js | comprensión operativa si son secundarios |
| 9 | LLM/agentes | diferenciador mediante un POC serio |

No estudiar React, PHP, Node y AWS en profundidad al mismo tiempo. Primero preguntar cuánto se usa cada uno.

## Angular + .NET como columna vertebral

Debes poder diseñar y explicar una feature completa:

```text
Angular
  -> autenticación y autorización
  -> API ASP.NET Core
  -> lógica de aplicación
  -> SQL Server/PostgreSQL
  -> integración externa o evento
  -> logs, métricas y trazas
  -> pipeline y despliegue en AWS
```

### Angular

- standalone components y lazy loading;
- TypeScript estricto;
- formularios tipados;
- Signals y RxJS con responsabilidades claras;
- interceptors, manejo consistente de errores y estados de carga;
- seguridad, rutas y permisos;
- pruebas de componentes y servicios;
- performance, accesibilidad y arquitectura por features.

Material: [[ANGULAR]], [[Angular moderno versiones y estrategia de upgrade]], [[Feature Architecture Boundaries]], [[Testing Strategy Angular]].

### .NET

- C# moderno y ASP.NET Core;
- diseño por capas o vertical slices sin dogmatismo;
- EF Core, transacciones y concurrencia;
- validación y Problem Details;
- OAuth 2.0/OIDC/JWT y autorización por policies;
- `HttpClientFactory`, timeout, retry y circuit breaker;
- pruebas unitarias, de integración y contract tests;
- configuración, secretos, health checks y observabilidad.

Material: [[ASP.NET Core APIs]], [[02 - HttpClientFactory Refit y gRPC]], [[03 - Resiliencia con Polly]], [[Integration Tests WebApplicationFactory]].

## PHP, React y Node.js sin dispersarte

### PHP

El vault ya contiene una ruta extensa de PHP/Laravel. Para esta postulación basta inicialmente con:

- sintaxis moderna, tipado, Composer y PSR;
- comprender routing, controllers, servicios, ORM y migrations;
- leer y modificar una API existente;
- auth, validación, errores y tests;
- queues, configuración, logs y despliegue;
- identificar deuda técnica en una aplicación legacy.

Material: [[LARAVEL]], [[APIs REST Laravel]], [[02_📚 Estudio/10_🔺 Laravel/13_ Senior Laravel/04 - Mantenimiento Legacy|Mantenimiento legacy en Laravel]], [[02 - Docker para Laravel]], [[03 - CI CD Laravel]].

Preguntar si “PHP” significa Laravel, Symfony, código propio o mantenimiento legacy. La estrategia cambia por completo.

### React

Como aparece una sola vez en las cinco vacantes, no se convierte todavía en una nueva especialidad. Si es secundario, apunta a:

- leer componentes funcionales y hooks;
- entender props, estado, efectos, routing y consumo de APIs;
- corregir una feature pequeña;
- reconocer problemas de renderizado, efectos y manejo de estado;
- ejecutar y ampliar pruebas existentes.

Si React ocupa más del 30-40 % del trabajo, entonces sí requiere una ruta separada antes de entrevista técnica.

### Node.js

Ya aparece en dos vacantes, pero solo en Cuidafarma es el núcleo. Aquí puede bastar con:

- event loop, async/await y manejo de errores;
- estructura de API y validación;
- consumo de servicios;
- configuración y secretos;
- tests, logs y Docker;
- poder leer un servicio Express o NestJS.

Material: [[NESTJS]], [[03 - Node.js Event Loop y Modulos]], [[Testing NestJS]].

## AWS, Docker y CI/CD

AWS aparece por segunda vez, por lo que pasa de vocabulario condicional a laboratorio demostrable. Antes de aprender muchos servicios, domina un flujo:

1. contenedor de Angular/.NET o frontend estático + API;
2. base administrada PostgreSQL o SQL Server según disponibilidad;
3. secretos fuera del repositorio;
4. logs, métricas y alarmas;
5. pipeline que prueba, construye y despliega;
6. health check, rollback y trazabilidad de versión.

Servicios concretos a confirmar: ECS/Fargate, Elastic Beanstalk, Lambda/API Gateway, RDS, S3, CloudFront, CloudWatch, Secrets Manager, ECR e IAM. No intentes cubrirlos todos en el CV sin experiencia.

Docker mínimo:

- Dockerfile multi-stage;
- usuario no root;
- `.dockerignore`;
- configuración por ambiente;
- health checks;
- Compose para desarrollo local;
- imágenes pequeñas, escaneo y tags inmutables.

CI/CD mínimo:

- lint, build y tests;
- artefacto o imagen versionada;
- gestión segura de credenciales;
- migraciones controladas;
- despliegue por ambiente;
- smoke test y rollback.

## Microservicios con criterio

Debes saber cuándo **no** elegirlos. Una buena respuesta de liderazgo incluye:

- límites de negocio y ownership;
- despliegue independiente como necesidad real;
- contratos y compatibilidad;
- datos por servicio y consistencia eventual;
- idempotencia, retries y mensajería;
- observabilidad distribuida;
- costo operativo, debugging y testing;
- estrategia de migración desde un monolito modular.

La palabra “microservicios” no justifica dividir la solución por controladores o por tecnología.

## Integración de LLM y agentes — diferenciador práctico

Aquí IA significa incorporar un modelo dentro de un producto o proceso, no conversar bien con ChatGPT.

La base práctica puede ser la **Responses API**, que acepta entradas de texto, imagen o archivo y permite resultados estructurados. También puede llamar código propio y herramientas integradas. Las herramientas disponibles se agrupan en built-in tools, MCP y funciones personalizadas con argumentos tipados: [referencia oficial de Responses API](https://developers.openai.com/api/reference/cli/resources/responses/methods/create).

### Qué aprender

- llamada server-side desde .NET o Node; nunca exponer la API key en Angular/React;
- instrucciones, contexto, límites de tokens, timeout y cancelación;
- salida estructurada para no depender de texto libre;
- function calling con schemas, validación, autorización y allowlists;
- estado del flujo, retries, idempotencia y presupuesto máximo;
- RAG o búsqueda de archivos solo cuando el caso requiera datos propios;
- protección ante prompt injection y fuga de datos;
- confirmación humana antes de ejecutar acciones sensibles;
- logging de latencia, tokens, costo, errores y llamadas de herramientas;
- datasets de prueba y regresiones de calidad.

OpenAI permite definir evaluaciones con criterios y una fuente de datos, y después ejecutar la evaluación con distintos modelos o parámetros. Ese enfoque es más sólido que aprobar una demo manual: [referencia oficial de Evals](https://developers.openai.com/api/reference/java/resources/evals/methods/create).

### POC recomendado

Construir un **asistente de incidencias** pequeño:

1. Angular recibe la descripción de un incidente.
2. .NET valida identidad, permisos y entrada.
3. El modelo clasifica severidad y devuelve JSON estructurado.
4. Una herramienta consulta documentación autorizada.
5. Otra herramienta prepara, pero no ejecuta, la creación de un ticket.
6. Un humano confirma la acción.
7. Se registran correlation ID, duración, tokens, costo y resultado.
8. Un dataset de casos mide clasificación correcta, uso indebido de herramientas y rechazo de instrucciones maliciosas.

Ese POC demuestra integración, agentes, seguridad, automatización, observabilidad y evaluación sin construir una plataforma innecesaria.

### Qué no priorizar todavía

- entrenar un modelo desde cero;
- un chatbot genérico sin integración;
- múltiples frameworks de agentes;
- bases vectoriales sin caso de uso;
- certificaciones de IA antes del proyecto;
- agentes autónomos con permisos amplios.

## Proyecto demostrable alineado

Extender el proyecto principal de la ruta:

- Angular como frontend principal.
- ASP.NET Core como API central.
- PostgreSQL o SQL Server.
- Docker Compose local.
- pipeline de pruebas/build/deploy.
- despliegue pequeño en AWS.
- un módulo PHP o Node únicamente como integración secundaria.
- asistente de incidencias con salida estructurada, una herramienta segura y evaluación.
- OpenAPI, ADR, logs/traces y runbook.

No añadir React al POC salvo que reclutamiento confirme que será parte importante del trabajo.

## Preparación de siete días

| Día | Foco | Entregable |
|---|---|---|
| 1 | CV y liderazgo | cuatro historias STAR con impacto medible |
| 2 | Angular + .NET | feature end to end y explicación arquitectónica |
| 3 | Datos e integración | PostgreSQL/SQL Server, índices, auth y resiliencia |
| 4 | Docker + CI/CD | build reproducible, pipeline y rollback explicado |
| 5 | AWS | despliegue pequeño con logs, secretos y health check |
| 6 | Stack secundario | lectura/cambio pequeño en PHP, Node o React según confirmación |
| 7 | IA + entrevista | POC mínimo y simulación de system design/liderazgo |

Postular antes de terminar los siete días.

## Preguntas probables de entrevista

### Liderazgo

- ¿Cómo decides entre entregar rápido y reducir deuda técnica?
- ¿Cómo manejas un desacuerdo de arquitectura?
- ¿Qué buscas en un code review?
- ¿Cómo acompañas a un desarrollador bloqueado?
- ¿Cómo comunicas un retraso o incidente al negocio?
- ¿Cómo mantienes estándares en varios stacks?

### Arquitectura y backend

- ¿Cuándo usarías un monolito modular en lugar de microservicios?
- ¿Cómo versionas una API consumida por Angular y terceros?
- ¿Cómo evitas duplicados cuando una operación se reintenta?
- ¿Cómo diagnosticas una integración lenta?
- ¿Cómo migrarías gradualmente un sistema PHP hacia .NET?
- ¿Qué diferencias relevantes encuentras entre SQL Server y PostgreSQL?

### Frontend

- ¿Cómo organizas un Angular grande?
- ¿Cómo decides entre Signals y RxJS?
- ¿Cómo compartirías componentes entre Angular y React sin acoplar productos?
- ¿Cómo aseguras permisos si ocultar un botón no es seguridad?

### Cloud y entrega

- Diseña un despliegue seguro en AWS.
- ¿Qué contiene tu pipeline?
- ¿Cómo haces rollback de código y base de datos?
- ¿Qué observas después de desplegar?
- ¿Qué problema resuelve Docker y cuál no?

### IA

- ¿Cuándo usar un flujo determinista frente a un agente?
- ¿Cómo evitas que el modelo ejecute acciones no autorizadas?
- ¿Cómo mides la calidad de una integración LLM?
- ¿Cómo controlas costo, latencia y datos sensibles?
- ¿Qué haces cuando el modelo devuelve una respuesta inválida?

## Palabras clave para el CV

`Technical Leadership`, `Angular`, `TypeScript`, `.NET`, `C#`, `ASP.NET Core`, `REST APIs`, `system integration`, `SQL Server`, `PostgreSQL`, `AWS`, `Docker`, `CI/CD`, `microservices`, `Scrum`, `code review`, `architecture decisions`, `incident response`, `observability`.

Añadir `PHP`, `Laravel`, `React`, `Node.js`, `LLM`, `AI agents` o `workflow automation` solo si puedes explicar experiencia o mostrar un proyecto, indicando con honestidad el nivel.

## Preguntas para reclutamiento

1. ¿Qué porcentaje del trabajo corresponde a Angular, .NET, PHP, React y Node.js?
2. ¿Cuál es el stack principal y cuál corresponde a mantenimiento legacy?
3. ¿PHP utiliza Laravel, Symfony u otro framework?
4. ¿Qué responsabilidades concretas tiene el “Líder Técnico”: arquitectura, mentoría, gestión de personas, estimación, releases u on-call?
5. ¿Cuántas personas tiene el equipo y cuántos reportes directos tendría?
6. ¿Qué servicios de AWS utilizan y quién administra la infraestructura?
7. ¿Los microservicios ya existen o se espera diseñar una modernización?
8. ¿Qué caso de uso de LLM/agentes están implementando y con qué proveedor?
9. ¿Cuántos días por semana se asiste a Miraflores?
10. ¿El rango salarial es negociable según el alcance real del liderazgo?

## Decisión final

Es una **postulación válida de prioridad media-alta** por la coincidencia Angular + .NET + datos + Docker, y porque refuerza AWS, PostgreSQL, CI/CD y microservicios dentro del radar. Puede convertirse en prioridad alta si el núcleo es Angular/.NET y el liderazgo tiene un alcance razonable.

No debe desencadenar cuatro rutas nuevas. React y PHP tienen una sola aparición; Node tiene dos pero solo una como núcleo; IA es un diferenciador opcional. La decisión correcta es postular, presentar profundidad en la columna vertebral y usar las preguntas de reclutamiento para decidir cuánto estudiar de las ramas secundarias.
