---
tags:
  - meta
  - estado
  - vault
Actualizado: 2026-05-22
---

# Estado del Vault - Contexto para futuras sesiones

Este archivo sirve como memoria tecnica del vault para continuar el trabajo en nuevas sesiones.

## Sobre Juber

- Desarrollador de software.
- Trabaja con .NET, Angular y tecnologias backend/frontend.
- Esta construyendo rutas de estudio junior a senior.
- Usa Obsidian como segundo cerebro para notas tecnicas, proyectos, carrera y vida personal.

## Estructura principal

```text
JUBER_OBSIDIAN/
  01_💼 Trabajo/
    01_🏢 NTT DATA/
    02_💼 Freelance/
  02_📚 Estudio/
    01_☁️ Azure/
    02_🟣 dotNET/
    03_☕ Java/
    04_🅰️ Angular/
    05_🏗️ Arquitectura/
    06_🎓 Cursos/
    07_⚙️ DevOps/
    08_🌿 Git/
    09_🇬🇧 Inglés/
  03_🚀 Carrera/
  04_🧠 Aprendizajes/
  05_🛠️ Proyectos/
  06_💰 Negocio/
  07_🏠 Personal/
  99_📦 Resources/
    01_🧩 Plantillas/
    02_📚 Referencias/
```

## Rutas principales

- Java: `02_📚 Estudio/03_☕ Java/JAVA.md`
- .NET: `02_📚 Estudio/02_🟣 dotNET/NET.md`
- Angular: `02_📚 Estudio/04_🅰️ Angular/ANGULAR.md`
- Git y GitHub: `02_📚 Estudio/08_🌿 Git/GIT.md`
- Plantillas: `99_📦 Resources/01_🧩 Plantillas/`
- Estado del vault: `99_📦 Resources/02_📚 Referencias/CLAUDE_ESTADO.md`

## Roadmaps creados

### Java

- Ruta: `02_📚 Estudio/03_☕ Java/JAVA.md`
- Estado: roadmap junior a senior con 148 notas Markdown.
- Modulos principales: fundamentos, intermedio, avanzado, build tools, testing, patrones, Spring, bases de datos, mensajeria, microservicios, arquitectura, DevOps, performance, senior y proyectos.

### .NET

- Ruta: `02_📚 Estudio/02_🟣 dotNET/NET.md`
- Estado: reconstruido el 2026-05-29 con criterio corregido similar a Angular.
- Estado actualizado el 2026-05-30 con una capa practica/senior adicional.
- Tiene 71 notas Markdown activas: 13 modulos principales, Home y roadmap.
- La version anterior quedo archivada dentro de `02_📚 Estudio/02_🟣 dotNET/98 - Archivo anterior/.NET anterior 2026-05-29/`.
- Criterio actual: menos notas, mas calidad; notas especificas por tema con ejemplos, decisiones, errores comunes y practica.
- Importante: no usar plantilla repetitiva para todas las notas. Cada nota debe tener secciones pensadas para el tema concreto.
- Modulos principales: inicio, C# y runtime, ASP.NET Core APIs, EF Core/SQL, seguridad/JWT, arquitectura, testing/calidad, performance/observabilidad, cloud/DevOps/mensajeria, senioridad/proyectos, laboratorios practicos, preguntas de entrevista y referencia senior.
- Mejora clave: se agregaron laboratorios progresivos, preguntas de entrevista, checklist profesional, ADRs, playbook de incidentes, guia de code review y proyecto capstone generico. No depende de StaffTrainingPlatform.

### Angular

- Ruta: `02_📚 Estudio/04_🅰️ Angular/ANGULAR.md`
- Estado: reconstruido desde cero el 2026-05-29 con criterio corregido por Juber.
- Estado actualizado el 2026-05-30 con una capa practica/senior adicional y mas detalle.
- Tiene 65 notas Markdown: 12 modulos principales, Home y roadmap.
- Correccion posterior: se detectaron notas demasiado cortas/casi vacias y se ampliaron. Validacion final: 65 notas activas, 0 notas bajo el umbral minimo revisado y 0 enlaces internos rotos.
- La version inflada/generica anterior fue eliminada a pedido de Juber.
- Criterio actual: menos notas, mas calidad; estilo similar a Java, con indices de modulo y notas especificas por tema.
- Importante: no usar plantilla repetitiva tipo "concepto / para que sirve / como funciona" en todas las notas. Cada nota debe tener secciones que sumen al tema concreto, con ejemplos, decisiones, errores comunes y practica cuando corresponda.
- Modulos principales: inicio, requisitos previos, TypeScript y CLI, componentes/templates/control flow, servicios/DI/HTTP, routing/forms/UX, RxJS/Signals/State, arquitectura/performance/seguridad, testing/ecosistema/senior, laboratorios practicos, preguntas de entrevista y referencia senior.
- Mejora clave: se agregaron laboratorios progresivos, preguntas de entrevista, checklist profesional, ADRs, guia de code review, playbooks de performance/migracion legacy y proyecto capstone Angular empresarial.

## Convenciones actuales

- Carpetas raiz: `NN_emoji Nombre`.
- Carpetas de primer nivel importantes: `NN_emoji Nombre`.
- Modulos tecnicos internos: `NN_ Nombre`, sin iconos para mantener rutas legibles.
- Indices principales usan `#Principal`.
- Notas usan frontmatter con `tags`, `Creado` y `Relacionado`.
- El avance se marca con checklists `- [ ]`.
- Se prefiere contenido real con ejemplos, tablas y errores comunes, no plantillas vacias.

## Historial

### 2026-05-21

- Se analizo el vault inicial.
- Se creo archivo de estado.
- Se comenzo la ruta Java.
- Se amplio Java a roadmap junior a senior.
- Se reconstruyo .NET como roadmap junior a senior.
- Se reconstruyo .NET nuevamente con 50 notas mas curadas, archivando la version anterior y eliminando el enfoque de muchas notas cortas.
- Se amplio .NET a 71 notas con laboratorios, entrevistas y referencia senior, ignorando StaffTrainingPlatform por pedido de Juber.

### 2026-05-22

- Se reorganizo el vault segun la estructura definida por Juber.
- Se movieron proyectos laborales a `01_💼 Trabajo/01_🏢 NTT DATA/01_📌 Proyectos/`.
- Se movio freelance a `01_💼 Trabajo/02_💼 Freelance/`.
- Se movieron plantillas a `99_📦 Resources/01_🧩 Plantillas/`.
- Se recuperaron y reubicaron notas antiguas de API, Arquitectura general y DevTalles dentro de la estructura nueva.
- Se creo `02_📚 Estudio/04_🅰️ Angular/` como roadmap Angular junior a senior.
- Se convirtio Angular a formato libro junior a senior con 103 notas.
- Se reemplazo Angular por una base de conocimiento con Home, roadmaps, plantillas y 678 notas nuevas tipo plantilla; lo anterior quedo archivado sin eliminarse.
- Se rellenaron las notas de Angular Knowledge Base con contenido compacto y ejemplos; no quedaron como plantillas vacias.
- Se borro y rehizo Angular desde cero con 84 notas, corrigiendo la version anterior por estar inflada/generica.
- Se borro y rehizo Angular nuevamente con 42 notas mas curadas, quitando la estructura repetitiva y priorizando contenido especifico por tema.
- Se amplio Angular a 65 notas con mas detalle practico: laboratorios, entrevistas, referencia senior, migracion legacy, performance y proyecto capstone.
- Se aplico diseño visual de carpetas con iconos y prefijos numericos.
- Se creo `02_📚 Estudio/08_🌿 Git/` como roadmap Git y GitHub junior a senior.

### StaffTrainingPlatform

- Se ordeno el proyecto `StaffTrainingPlatform` dentro de Freelance/Sebastian - Apolo.
- Se documento la decision `ADR-001 - Autenticacion JWT y API Multi-Tenant` para CapaActivaCorp.
- Se definio JWT, contextos `platform` y `tenant`, endpoints, claims, naming de controllers/requests y reglas de API multi-tenant.
- Se documento la decision `ADR-002 - Manejo de Access Token y Refresh Token`: access token 15 minutos, refresh token 7 dias, cliente reacciona al 401, refresh token persistido para revocacion.
- Se documento la decision `ADR-003 - Seguridad Avanzada de JWT y Refresh Tokens`: rotation con deteccion de reuso, `ReplacedByToken`, contexto de login, aprobacion por email, rate limiting y senales sospechosas.

### Laravel

- Se creo `02_📚 Estudio/10_🔺 Laravel/` como libro Laravel junior a senior con 88 notas.
- Ruta principal: `02_📚 Estudio/10_🔺 Laravel/LARAVEL.md`.
- La intencion correcta no es solo roadmap, sino material de estudio tipo libro: capitulos, explicacion, modelo mental, ejemplos, practica, preguntas de repaso y criterios junior/mid/senior.
- Modulos principales: fundamentos PHP y web, Laravel Core, Eloquent, APIs REST, auth y seguridad, testing, jobs/queues/eventos, arquitectura, frontend con Laravel, DevOps, performance, ecosistema/calidad, senior y proyectos.

## Siguiente paso recomendado

Validar en Obsidian que las rutas principales se vean bien:

- `02_📚 Estudio/03_☕ Java/JAVA.md`
- `02_📚 Estudio/02_🟣 dotNET/NET.md`
- `02_📚 Estudio/04_🅰️ Angular/ANGULAR.md`
