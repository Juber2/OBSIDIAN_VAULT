---
tags:
  - Postulaciones
  - CV
  - Diagnóstico
Creado: 2026-08-31
Relacionado:
  - 16 - Radar acumulativo de vacantes
  - 22 - Inventario de Ejemplos Propios
Fuente: CV JUBER GUTIERREZ ORE.pdf
---

# Mi CV frente a las 14 vacantes

Análisis del CV real (`C:\JUBER\CV\CV JUBER GUTIERREZ ORE.pdf`) contra lo que exigen las vacantes de [[16 - Radar acumulativo de vacantes]]. Escrito el 31/08/2026.

## Perfil real, según el CV

| Dato | Valor |
|---|---|
| Título actual | Analista desarrollador de software |
| Experiencia declarada | Más de 4 años (Nov 2021 en adelante, sin contar prácticas) |
| Experiencia total con prácticas | ~6.9 años (desde oct 2019) |
| Eje técnico dominante | **.NET (C#) + SQL Server** — presente en las 4 experiencias |
| Formación | Ing. de Sistemas **en curso** (ciclo 8) + Técnico titulado |

## ⚠️ La corrección más importante: Java

Las notas [[RUTA FULL STACK ANGULAR NET AZURE]] y [[06 - Ruta Java Spring Boot como segundo eje backend]] plantean Java/Spring como **"segundo eje backend al mismo nivel que .NET"**. Con el CV a la vista, eso era optimista y hay que corregirlo:

> **El CV no tiene ni una línea de Java backend.** La única aparición de Java es *"Aplicaciones móviles Android (Java/Kotlin + Jetpack Compose)"* — que es Java, pero no es Spring, ni APIs, ni backend empresarial.

Consecuencia práctica:

| | Realidad |
|---|---|
| **.NET** | Demostrable con 4 experiencias y ~5 años. Es tu eje real. |
| **Java/Spring** | **Aspiracional.** Las 7 vacantes que lo piden suelen pedir 3-5 años de Java/Spring que el CV no puede sostener. |

Esto **no significa abandonar Java** — significa cambiar la estrategia:

- **Prioridad de postulación:** las vacantes .NET primero (Entelgy, TSOFT, TCS, Periferia). Ahí competís con evidencia, no con intención.
- **Java sigue en el plan de estudio**, pero como inversión a mediano plazo y para vacantes **políglotas** (Stefanini, Backend Java+.NET), donde .NET es la mitad del requisito y Java el complemento.
- **Nunca afirmar experiencia en Java que no existe.** Se cae en la primera pregunta técnica y arrastra la credibilidad de toda la entrevista. La formulación honesta y aceptada es: *"Mi experiencia profesional es .NET; vengo formándome en Java/Spring y sé mapear los conceptos entre ambos ecosistemas"* — y ahí la tabla de equivalencias de la nota 17 sí juega a favor.

## 💎 Fortalezas que el CV tiene pero no comunica bien

Estas ya las hiciste; están escritas de forma que un reclutador no las detecta:

| Lo que dice el CV | Lo que en realidad es | Vacantes donde vale |
|---|---|---|
| *"Integración de pagos mediante QR (Izipay)"* | **Integración con un tercero que no controlás** — contratos, errores, resiliencia | REST/integraciones: 10/14 |
| *"Automatización de procesos... (proyecto BBVA)"* | **Experiencia en sector financiero/banca** | Synopsis (filtro excluyente), Java BIAN, Coforge/BCP |
| *"migrando de un esquema manual en IIS a despliegues continuos"* | Historia completa de **transformación DevOps** con antes y después | CI/CD: 7/14 |
| *"Optimización de consultas y procedimientos... reduciendo tiempos de respuesta"* | Respuesta directa a **la pregunta #2 más frecuente** del banco | Consulta lenta: 5 vacantes |
| *"Migración de módulos legacy a arquitectura moderna"* | **Modernización incremental** sin romper lo existente | Periferia, Lider Técnico |
| *"procesos batch y ETLs para manejo de datos masivos"* | Procesamiento de alto volumen | Alta concurrencia, Data Factory |
| *"Clean Architecture, CQRS y MediatR"* | Arquitectura aplicada, no teórica | Clean/DDD/CQRS: 3/14 |

> **Izipay + BBVA es tu evidencia de sector financiero.** [[Synopsis - Backend Senior Financiero]] trata el dominio financiero como filtro casi excluyente, y vos lo tenés — pero enterrado en una línea sobre Power Automate. Eso hay que subirlo.

## 🕳️ Lo que falta en el CV (palabras que el radar exige y no aparecen)

Ausencias que cuestan filtros ATS y credibilidad, ordenadas por frecuencia en el radar:

| Ausente en el CV | Lo piden | ¿Lo tenés? |
|---|---:|---|
| **Testing / pruebas unitarias** | 6/14 | No aparece **ninguna** mención en todo el CV — es el hueco más visible |
| **Microservicios** | 7/14 | No aparece, aunque trabajás con APIs y Clean Architecture |
| **Mensajería / colas / eventos** | 4/14 | No aparece |
| **Observabilidad / monitoreo** | 4/14 | Solo *"registro de logs"*, muy débil |
| **Kubernetes** | 2/14 | No aparece (tenés Docker, falta el siguiente paso) |
| **Git / control de versiones** | 6/14 | Solo *"GitHub"* suelto en Herramientas, sin contexto de PRs o code review |
| **Seguridad / JWT / OAuth** | 7/14 | No aparece |
| **Inglés** | Filtro en 1 | No aparece — asumir que es un filtro real donde lo piden |

Además, **ninguna métrica está cuantificada**: *"reduciendo tiempos de respuesta"* sin un número no convence. Si tenés el dato, ponelo; si no, describí el alcance (*"en módulos consultados por N usuarios"*).

## Encaje realista por vacante

Ordenado por dónde conviene invertir el esfuerzo primero:

| Vacante | Encaje | Por qué |
|---|---|---|
| [[Entelgy - Backend NET Azure]] | 🟢 **Fuerte** | .NET, SQL Server, Azure, Docker, CI/CD, Clean Architecture: todo con evidencia. Faltan Dapper, Service Bus y testing |
| [[TCS - NET Azure Android]] | 🟢 **Fuerte** | .NET + Azure **y además Android real** (Java/Kotlin + Jetpack Compose) — combinación poco común, aprovechala |
| [[TSOFT - NET Angular Oracle]] | 🟢 **Fuerte** | .NET + Angular + Oracle listado; falta profundidad en PL/SQL |
| [[Periferia - Migracion VB6 NET 10 WinForms]] | 🟢 **Fuerte** | Tenés **VB, WPF, Web Forms** y un proyecto real en Visual Basic. Encaja mejor de lo que la nota asumía |
| [[Synopsis - Backend Senior Financiero]] | 🟡 Media-alta | Backend .NET sí + **evidencia financiera real (BBVA, Izipay)**. Falta SOAP/WSDL |
| [[Lider Tecnico Full Stack Angular NET]] | 🟡 Media | Angular + .NET + **Laravel real**. El filtro es la evidencia de liderazgo, no el stack |
| [[Stefanini - Java NET Angular]] | 🟡 Media | .NET y Angular sí; Java es la mitad faltante |
| [[Backend Java NET DDD DB2]] | 🟡 Media | .NET + DDD/CQRS sí; Java y DB2 no |
| [[Full Stack Senior Java Angular AWS]] | 🔴 Stretch | Java no, AWS no, y 5 años de APIs es limítrofe |
| [[ISP Consulting - Java Angular GCP]] | 🔴 Stretch | Java no, GCP no |
| [[Full Stack Java React San Isidro]] | 🔴 Stretch | Java no, React no, inglés no evidenciado |
| [[Java Senior BIAN OpenShift Azure APIM]] | 🔴 Stretch | Java no, BIAN no, OpenShift no |
| [[Postulacion\|Coforge/BCP Senior Java]] | 🔴 Stretch | Todo el estudio está hecho, pero pide **Senior Java** sin experiencia Java en el CV |
| [[Cuidafarma - Backend Node AWS]] | 🔴 Stretch | Node y AWS como núcleo, ninguno en el CV |

**4 vacantes fuertes, 4 medias, 6 stretch.** Postular a todas sigue siendo correcto (regla del 70%), pero el esfuerzo de adaptación de CV se concentra en las 8 primeras.

## Acciones concretas sobre el CV

Ordenadas por impacto:

- [ ] **Cambiar el título** según la vacante: "Analista desarrollador de software" no matchea búsquedas. Usar *Desarrollador Backend .NET* / *Full Stack .NET + Angular* / *Desarrollador .NET | Azure* según el anuncio.
- [ ] **Agregar una línea de testing** en la experiencia de Pluxee o en el proyecto freelance — es el hueco más visible y aparece en 6 vacantes.
- [ ] **Subir Izipay y BBVA** a un lugar visible: son integración con terceros y sector financiero, no detalles menores.
- [ ] **Cuantificar al menos 2 logros** con números reales.
- [ ] **Agregar seguridad** (JWT/autenticación) si la implementaste en Pluxee o en el sistema de capacitación.
- [ ] **Enriquecer Git**: no basta con listar "GitHub" — mencionar PRs, code review o branching en la experiencia.
- [ ] **Agregar links** visibles a GitHub y LinkedIn en el encabezado.
- [ ] **Mencionar el laboratorio** ([[06 - Laboratorio Integrado - API de Ordenes]]) en "Proyectos" cuando esté desplegado — ahí entran microservicios, mensajería, observabilidad y Kubernetes de forma honesta.
- [ ] **Definir cómo declarar el inglés**: si es básico, no mencionarlo; si es intermedio lector, decirlo con precisión.

## Lo que esto cambia en el plan de estudio

El plan de [[04 - Plan Detallado 3 Semanas (dia a dia)]] sigue siendo válido, con dos ajustes de énfasis:

1. **Testing sube de prioridad.** Es la ausencia más visible del CV y aparece en 6 vacantes. Está en el sábado 5 de septiembre (fase 1 del laboratorio) — que ese bloque no se salte.
2. **El laboratorio deja de ser opcional.** Es la única fuente de evidencia para microservicios, mensajería, observabilidad, seguridad moderna y Kubernetes — cinco huecos del CV a la vez. Cada fase que completás es una línea nueva y honesta en el CV.
