---
tags:
  - Postulacion
  - TSOFT
  - FullStack
  - dotNET
  - Angular
  - Oracle
  - PLSQL
Creado: 2026-08-30
Estado: Por postular
Empresa: TSOFT
Modalidad: No indicada
Ubicacion: No indicada
Sueldo: No indicado
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - NET
  - ANGULAR
---

# TSOFT — Full Stack .NET + Angular + Oracle

## Veredicto rápido

**Postular: sí, prioridad alta.**

Es una de las vacantes con mejor alineación directa con la ruta objetivo: .NET/C#, Angular, APIs REST, integración end to end, testing, seguridad, rendimiento y despliegues. La especialización diferencial es Oracle con PL/SQL.

El filtro principal es acreditar tres años desarrollando aplicaciones web y experiencia comprobable en los tres componentes: frontend, backend y base de datos. Si .NET y Angular son sólidos pero Oracle es una brecha, sigue siendo razonable postular y preparar un laboratorio intensivo.

## Encaje estimado

| Escenario | Encaje aproximado |
|---|---:|
| 3+ años con .NET, Angular y Oracle/PLSQL | 85-95 % |
| .NET y Angular sólidos; SQL fuerte pero Oracle limitado | 70-85 % |
| Full Stack sólido con otra base relacional y laboratorio Oracle | 65-78 % |
| Angular o .NET principalmente académico | Riesgo alto |
| Sin experiencia escribiendo SQL complejo | Riesgo alto por peso de base de datos |

## Perfil real

Es un Full Stack empresarial clásico con mucho peso de base de datos:

```text
Angular + TypeScript
        │
        ▼
ASP.NET Core / C# / REST APIs
        │
        ▼
Oracle Database
  ├── SQL
  ├── PL/SQL
  ├── procedures/functions
  ├── packages/triggers
  └── índices y planes de ejecución
```

No parece un puesto cloud ni DevOps. El foco es desarrollar, mantener, corregir, optimizar y desplegar aplicaciones empresariales existentes.

## Matriz de requisitos

| Capacidad | Importancia | Evidencia necesaria |
|---|---|---|
| .NET/C# | Crítica | APIs, lógica, integración, errores y rendimiento |
| Angular/TypeScript | Crítica | feature completa, arquitectura y consumo de APIs |
| Oracle SQL | Crítica | consultas, índices, ejecución y optimización |
| PL/SQL | Crítica | procedures, functions, packages y triggers |
| APIs REST | Alta | contratos, seguridad, validación y errores |
| Integración end to end | Alta | frontend, backend y base funcionando juntos |
| Mantenimiento evolutivo/correctivo | Alta | debugging, impacto, regresión y entrega segura |
| Testing y QA | Alta | pruebas antes de entregar y corrección reproducible |
| Despliegues QA/producción | Alta | checklist, configuración, migraciones y rollback |
| Seguridad/performance | Alta | prácticas aplicadas, no definiciones |
| Seguros/alta concurrencia | Deseable | evidencia sectorial o diseño razonado |
| IA para desarrollo | Deseable | flujo verificable y revisión humana |

## .NET y C#

Debes poder demostrar:

- C# moderno, POO y principios SOLID con criterio;
- ASP.NET Core Web APIs;
- DI, middleware y configuración;
- DTOs, validación y Problem Details;
- autenticación/autorización;
- acceso a Oracle desde .NET;
- transacciones y manejo de concurrencia;
- consumo de APIs con timeout y cancelación;
- logs estructurados y diagnóstico;
- pruebas unitarias y de integración;
- performance y uso correcto de recursos.

Material: [[ASP.NET Core APIs]], [[02 - HttpClientFactory Refit y gRPC]], [[03 - Resiliencia con Polly]], [[Integration Tests WebApplicationFactory]].

## Angular

Preparar una feature empresarial completa:

- componentes standalone;
- TypeScript estricto;
- routing y lazy loading;
- formularios tipados y validación;
- HttpClient e interceptors;
- Signals y RxJS;
- auth, permisos y guards;
- manejo de carga, vacío y error;
- HTML semántico y CSS responsive;
- pruebas de componentes y servicios;
- performance y accesibilidad.

Material: [[ANGULAR]], [[Angular moderno versiones y estrategia de upgrade]], [[Feature Architecture Boundaries]], [[Testing Strategy Angular]].

## Oracle SQL — prioridad específica

Oracle llega a tres apariciones en el radar, por lo que deja de ser una brecha aislada. Debes practicar:

- tipos `NUMBER`, `VARCHAR2`, `DATE`, `TIMESTAMP`, `CLOB` y `BLOB`;
- joins, subqueries, CTE y operaciones de conjuntos;
- funciones analíticas;
- sequences e identity;
- índices simples, compuestos y selectividad;
- planes de ejecución;
- estadísticas del optimizador;
- binds y prevención de SQL injection;
- paginación;
- transacciones, locks, deadlocks y aislamiento;
- procesamiento eficiente de volúmenes altos;
- conexión, pooling y timeout desde .NET.

No basta trasladar sintaxis de SQL Server. Debes reconocer diferencias de dialecto y comportamiento.

## PL/SQL

### Procedures y functions

- parámetros `IN`, `OUT` e `IN OUT`;
- tipos y retorno;
- manejo de excepciones;
- cursores cuando estén justificados;
- transacciones y responsabilidad de `COMMIT`;
- pruebas y diagnóstico.

### Packages

- specification y body;
- encapsulación de operaciones relacionadas;
- dependencias y recompilación;
- estado de sesión y riesgos;
- versionado y despliegue.

### Triggers

- `BEFORE`/`AFTER` e insert/update/delete;
- nivel fila frente a statement;
- auditoría y reglas simples;
- mutating table y efectos ocultos;
- cuándo reemplazarlos por lógica explícita.

### Rendimiento PL/SQL

- evitar procesamiento fila por fila innecesario;
- operaciones set-based;
- bulk collect/forall cuando corresponda;
- planes y estadísticas;
- medir antes de optimizar.

## Integración .NET ↔ Oracle

Preparar:

- driver/proveedor Oracle para .NET;
- parámetros tipados y consultas parametrizadas;
- mapeo de tipos Oracle/C#;
- llamadas a procedures y functions;
- manejo de REF CURSOR si el proyecto lo usa;
- connection pooling;
- transacciones locales;
- timeout y cancelación;
- logging sin exponer datos sensibles;
- repositorios o acceso directo según complejidad;
- migrations/scripts versionados.

Preguntar si utilizan EF Core, Dapper, ODP.NET directo o una capa propia.

## Mantenimiento evolutivo y correctivo

Debes preparar historias reales sobre:

- reproducir un error reportado por QA;
- rastrear frontend → API → Oracle;
- identificar causa raíz;
- realizar el cambio mínimo seguro;
- evaluar impacto y regresión;
- añadir una prueba que evite recurrencia;
- documentar la solución;
- desplegar y verificar en QA/producción.

En aplicaciones legacy, primero se caracteriza el comportamiento existente. No se refactoriza una zona crítica sin cobertura o estrategia de rollback.

## Alta concurrencia y rendimiento

### Backend

- endpoints async cuando existe I/O;
- evitar bloqueos y estado compartido;
- pool de conexiones dimensionado;
- timeouts y límites;
- caché solo con estrategia de invalidación;
- paginación y streaming;
- métricas de latencia, errores y saturación.

### Oracle

- índices según filtros y joins reales;
- planes de ejecución;
- bind variables;
- locks y transacciones cortas;
- evitar N+1 y consultas repetidas;
- batch y operaciones set-based;
- detección de consultas caras.

### Angular

- evitar solicitudes duplicadas;
- lazy loading;
- renderizado y change detection controlados;
- paginación/virtualización en grandes volúmenes;
- bundle y recursos optimizados.

## Seguridad

- validación en backend;
- autorización del lado servidor;
- consultas parametrizadas;
- protección de secretos;
- mínimo privilegio para usuario de Oracle;
- no registrar credenciales ni datos sensibles;
- manejo seguro de errores;
- dependencias actualizadas;
- auditoría de acciones sensibles;
- protección ante XSS/CSRF según arquitectura.

## Pruebas y QA

- unit tests para reglas;
- integration tests de API;
- pruebas de integración con Oracle o ambiente equivalente;
- contract tests del frontend/backend;
- regresión para mantenimiento correctivo;
- pruebas de procedures/functions críticas;
- smoke tests tras despliegue;
- pruebas básicas de carga para flujos de alta concurrencia.

Material: [[Testing Strategy Angular]], [[Integration Tests WebApplicationFactory]], [[09 - Seguridad testing y calidad]].

## Despliegues QA y producción

Preparar un checklist:

1. versión y artefacto identificables;
2. configuración por ambiente;
3. scripts Oracle revisados y ordenados;
4. backup o estrategia de recuperación;
5. compatibilidad entre API y esquema;
6. smoke tests;
7. monitoreo posterior;
8. rollback de aplicación y plan para base de datos;
9. comunicación de resultado e incidencias.

## IA aplicada al desarrollo

En esta vacante IA es una herramienta de productividad, no una funcionalidad del producto.

Usos defendibles:

- generar un primer borrador de código;
- proponer casos de prueba;
- explicar código legacy;
- sugerir refactorizaciones;
- crear documentación inicial;
- analizar un stack trace o consulta;
- comparar opciones de optimización.

Controles necesarios:

- revisar cada cambio;
- ejecutar tests y análisis estático;
- no compartir código o datos confidenciales sin autorización;
- verificar APIs y versiones;
- no aceptar una optimización sin medir;
- mantener responsabilidad humana sobre el resultado.

No añadir “IA” al CV únicamente por usar un chat. Preparar un ejemplo concreto de ahorro de tiempo con calidad verificable.

## Proyecto demostrable

Crear un módulo de gestión de pólizas o solicitudes:

- Angular con listado, filtros, formulario y detalle;
- ASP.NET Core REST API;
- Oracle como base;
- procedure para una operación compleja justificada;
- package para operaciones relacionadas;
- trigger únicamente para auditoría si es adecuado;
- consultas con índices y planes comparados;
- autenticación y permisos;
- unit/integration tests;
- logs y medición de rendimiento;
- despliegue simulado QA → producción;
- uso documentado de IA con revisión y pruebas.

## Preparación de siete días

| Día | Foco | Entregable |
|---|---|---|
| 1 | CV/evidencia | historias .NET, Angular, SQL y mantenimiento |
| 2 | Oracle SQL | consultas, índices y planes de ejecución |
| 3 | PL/SQL | procedure, function, package y trigger |
| 4 | .NET + Oracle | API, pooling, transacción y pruebas |
| 5 | Angular | feature integrada y manejo de errores |
| 6 | Performance/QA | carga, consulta lenta, regresión y RCA |
| 7 | Entrevista | demo end to end y simulación técnica |

Postular antes de completar el plan.

## Preguntas probables

- ¿Cómo integras .NET con Oracle?
- ¿Cuándo usar un procedure en vez de lógica C#?
- ¿Qué diferencia existe entre procedure, function y package?
- ¿Qué riesgos tienen los triggers?
- ¿Cómo diagnosticas una consulta lenta?
- ¿Cómo interpretas un plan de ejecución?
- ¿Cómo investigas un bloqueo o deadlock?
- ¿Cómo corriges un incidente sin romper funcionalidad existente?
- ¿Cómo pruebas una integración Angular/.NET/Oracle?
- ¿Cómo realizarías un despliegue con cambios de base de datos?
- ¿Cómo diseñarías para alta concurrencia?
- ¿Cómo utilizas IA sin comprometer seguridad ni calidad?

## Palabras clave para el CV

`.NET`, `C#`, `ASP.NET Core`, `Angular`, `TypeScript`, `REST APIs`, `Oracle Database`, `SQL`, `PL/SQL`, `stored procedures`, `functions`, `packages`, `triggers`, `query optimization`, `execution plans`, `indexes`, `application maintenance`, `incident resolution`, `unit testing`, `integration testing`, `security`, `performance`.

## Preguntas para reclutamiento

1. ¿Qué versiones de .NET, Angular y Oracle utilizan?
2. ¿Qué porcentaje del trabajo corresponde a frontend, backend y PL/SQL?
3. ¿Los tres componentes requieren experiencia profesional previa?
4. ¿Utilizan EF Core, Dapper u ODP.NET directo?
5. ¿Cuánto del sistema es mantenimiento y cuánto desarrollo nuevo?
6. ¿El cliente pertenece al sector seguros?
7. ¿Cuál es la modalidad, sede y frecuencia presencial?
8. ¿Cuál es el rango salarial y tipo de contrato?
9. ¿Existe soporte fuera de horario?
10. ¿Cómo gestionan scripts y rollback de Oracle?

## Decisión final

TSOFT debe estar entre las postulaciones principales por su coincidencia directa con Angular + .NET. Oracle ya aparece en tres vacantes y debe convertirse en un laboratorio concreto de la ruta.

La preparación diferencial es **Oracle SQL/PLSQL y mantenimiento empresarial**, no aprender un nuevo stack.
