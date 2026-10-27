---
tags:
  - Postulacion
  - TCS
  - dotNET
  - Azure
  - Android
Creado: 2026-08-30
Estado: Por postular
Empresa: Tata Consultancy Services
Modalidad: Híbrido
Ubicación: Lima
Sueldo: S/ 5,000 - S/ 6,000
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - NET
  - Azure
---

# TCS — Desarrollo .NET, Azure y Android

## Veredicto rápido

**Postular: sí, con una aclaración previa o durante el primer contacto.**

El puesto coincide con tu eje .NET + Azure, tiene requisitos básicos/intermedios y el rango salarial está explícito. Sin embargo, **no es una vacante Full Stack Angular**: las funciones incluyen construcción y mantenimiento de aplicaciones Android. Antes de dedicar tiempo importante a Kotlin/Android, hay que confirmar cuánto trabajo móvil existe realmente.

## Encaje estimado

| Dimensión | Encaje | Observación |
|---|---|---|
| .NET/C# y APIs REST | 🟢 Alto | Es parte central de tu ruta y vault |
| REST y JSON | 🟢 Alto | Fundamento ya cubierto |
| Git | 🟢 Alto | Existe ruta completa [[GIT]] |
| Linux | 🟢 Alto/medio | Existe libro [[LINUX]]; validar práctica real |
| Azure App Service/Functions/Storage | 🟡 Medio | Material cubierto; falta demostrar despliegue |
| Azure Pipelines | 🟡 Medio/bajo | CI/CD transferible, pero conviene practicar el producto |
| API Management | 🟡 Medio | Concepto cubierto; preparar políticas y casos de uso |
| Scrum/Jira/Confluence | 🟡 Básico | Brecha pequeña y rápida de cerrar |
| Android | 🔴 No evidenciado | No existe evidencia Android en el vault |
| Angular | ⚪ No solicitado | No mejora directamente esta candidatura |

**Encaje documental aproximado: 65-75 %.** El rango depende de si Android es apoyo ocasional o responsabilidad diaria. “No evidenciado” significa que el vault no demuestra esa competencia; no presupone que nunca la hayas usado.

## Inconsistencia importante del anuncio

Android aparece en las **funciones**, pero no se especifica en los **requisitos**:

- lenguaje: Kotlin o Java;
- UI: Jetpack Compose o XML Views;
- arquitectura: MVVM/MVI;
- librerías y nivel esperado;
- publicación, testing o CI móvil.

Esto puede significar que el anuncio está incompleto, que el rol apoya a un equipo móvil o que buscan un perfil móvil con backend. No conviene adivinar.

## Preguntas para reclutamiento

1. ¿Qué porcentaje del trabajo es Android y qué porcentaje .NET/Azure?
2. ¿La experiencia Android es obligatoria o se espera aprenderla dentro del equipo?
3. ¿Utilizan Kotlin, Java, Jetpack Compose o vistas XML?
4. ¿El backend es ASP.NET Core y qué versión utilizan?
5. ¿Azure Pipelines se usa para backend, Android o ambos?
6. ¿Qué frecuencia presencial exige la modalidad híbrida?
7. ¿Existe soporte productivo, guardias u on-call?
8. ¿El rango de S/ 5,000-6,000 es bruto y qué beneficios incluye?

## Requisitos traducidos a capacidades

| Requisito del anuncio | Qué debes poder demostrar | Recurso |
|---|---|---|
| C#/.NET básico-intermedio | API REST con DI, validación, errores y tests | [[ASP.NET Core APIs]] |
| REST/JSON | verbos, códigos, contratos, serialización y errores | [[Controllers Minimal APIs Routing]] |
| Git | branch, commit, pull request, conflictos y revisión | [[GIT]] |
| Linux | navegación, procesos, logs, permisos, red y variables | [[LINUX]] |
| App Service y Functions | desplegar API y función; configuración por ambiente | [[03 - App Service y Azure Functions]] |
| Storage | Blob/Queue, acceso seguro e integración desde .NET | [[02 - Azure SQL Storage y Key Vault]] |
| API Management | gateway, policies, auth, rate limit y versionado | [[04 - APIM y Application Gateway]] |
| Pipelines | build, test, artefacto y deploy | [[03 - CI CD para .NET]] |
| Scrum | roles, eventos, artefactos y trabajo incremental | Preparación breve en esta nota |
| Jira/Confluence | navegar tickets, estados, comentarios y documentación | Práctica de usuario |
| Android | confirmar alcance antes de crear una ruta completa | Brecha condicionada |

## Qué aprender o repasar para esta vacante

### P0 — Antes de una entrevista

- [ ] Crear una API ASP.NET Core pequeña con REST y JSON.
- [ ] Explicar middleware, DI, validación y `ProblemDetails`.
- [ ] Desplegar una API en App Service.
- [ ] Crear una Azure Function HTTP o disparada por cola.
- [ ] Guardar/leer un archivo en Blob Storage.
- [ ] Explicar APIM: policies, autenticación, rate limiting y transformación.
- [ ] Leer un pipeline YAML y explicar sus etapas.
- [ ] Repasar comandos Linux de diagnóstico.
- [ ] Explicar Scrum sin recitar un glosario.

### P1 — Si confirman responsabilidad Android

- Kotlin esencial y null safety.
- Android Studio, Gradle y estructura del proyecto.
- Activity/Fragment a nivel de mantenimiento.
- Jetpack Compose si es el stack actual del cliente.
- ViewModel, estado, navegación y ciclo de vida.
- Coroutines y Flow.
- Retrofit/OkHttp, JSON y consumo de APIs.
- almacenamiento seguro de tokens;
- pruebas unitarias y UI básicas;
- build variants, signing y pipeline.

No crear todavía un libro completo de Android. Primero confirmar que el requisito es real y conocer el stack exacto.

## Preparación intensiva de tres días

### Día 1 — .NET y REST

- Construir/revisar API CRUD.
- Validación y errores consistentes.
- JSON, status codes, paginación y OpenAPI.
- Una prueba de integración.

### Día 2 — Azure

- App Service, Functions y Blob Storage.
- APIM y flujo de una petición.
- Pipeline build/test/deploy.
- Configuración, secretos y managed identity.

### Día 3 — Operación y colaboración

- Linux: `ls`, `cd`, `grep`, `curl`, `ps`, `top`, `journalctl`, permisos y variables.
- Git: branch, merge/rebase conceptual, conflictos y PR.
- Scrum, Jira y Confluence.
- Android: solo panorama técnico para conversar con honestidad.

## Scrum, Jira y Confluence — mínimo suficiente

### Scrum

- Product Owner prioriza valor y Product Backlog.
- Scrum Master facilita y ayuda a remover impedimentos.
- Developers construyen un incremento usable.
- Eventos: Sprint, Planning, Daily, Review y Retrospective.
- No confundir Daily con reporte individual al jefe.

### Jira

- Buscar y actualizar issue.
- Entender epic, story, task, bug y subtask según la convención del equipo.
- Mover estados, comentar, adjuntar evidencia y relacionar tickets.
- Interpretar backlog, sprint y board.

### Confluence

- Buscar documentación y navegar espacios.
- Crear/editar páginas y usar plantillas.
- Enlazar decisiones, tickets y runbooks.
- Registrar fecha, responsable y contexto para evitar documentación huérfana.

## Preguntas técnicas probables

### .NET/REST

- ¿Cómo diseñarías un endpoint para subir metadata y un archivo?
- ¿Qué diferencia `401` de `403`?
- ¿Cómo manejas excepciones globalmente?
- ¿Cómo consumes un servicio externo de forma resiliente?
- ¿Cómo serializa ASP.NET Core un objeto a JSON?

### Azure

- ¿Cuándo usar App Service y cuándo Functions?
- ¿Cómo accede una app a Blob Storage sin guardar credenciales?
- ¿Qué aporta API Management delante de una API?
- ¿Qué etapas tendría un pipeline?
- ¿Dónde revisarías logs si el despliegue funciona pero la API responde 500?

### Linux/Git/Agile

- ¿Cómo encontrarías un proceso o revisarías logs?
- ¿Cómo resolverías un conflicto Git?
- ¿Qué haces si una historia no estará terminada al cierre del sprint?
- ¿Cómo comunicas un incidente a QA, cloud y negocio?

## Palabras clave para adaptar el CV

`C#`, `ASP.NET Core`, `.NET`, `REST APIs`, `JSON`, `Git`, `Linux`, `Azure App Service`, `Azure Functions`, `Azure Storage`, `API Management`, `CI/CD`, `Azure Pipelines`, `Scrum`, `Jira`, `Confluence`.

Usar Android/Kotlin únicamente si existe experiencia real o después de completar una práctica demostrable.

## Evidencia mínima para mejorar el encaje

- [ ] API .NET desplegada en App Service.
- [ ] Function integrada con Blob o Queue Storage.
- [ ] Pipeline YAML con build, test y deploy.
- [ ] APIM delante de la API con una policy sencilla.
- [ ] README con comandos Linux de diagnóstico y runbook.
- [ ] Respuesta confirmada sobre el peso real de Android.

## Decisión final

**Conviene postular ahora** si aceptas el rango, la modalidad y la posibilidad de tocar Android. No retrases la candidatura para aprender Android desde cero. Declara con precisión tu fortaleza en .NET/Azure y pregunta si el componente móvil admite curva de aprendizaje.

