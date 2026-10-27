---
tags:
  - Postulacion
  - FullStack
  - Java
  - SpringBoot
  - React
  - Ingles
Creado: 2026-08-30
Estado: Por evaluar
Empresa: No indicada
Modalidad: Híbrido
Ubicacion: San Isidro, Lima
Sueldo: S/8,000-S/9,000
Seniority: Senior
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - JAVA
---

# Full Stack Java + React — San Isidro

## Veredicto rápido

**Postular si puedes acreditar cinco años Full Stack y sostener una entrevista en inglés. Prioridad media-alta.**

El salario es coherente con una posición senior y superior a la mayoría de las vacantes analizadas. El núcleo es Java/Spring Boot + React; las principales condiciones documentales son cinco años Full Stack e inglés intermedio/avanzado.

No coincide con Angular/.NET, pero Java ya es una segunda vía consolidada y React alcanza su segunda aparición. Por ello merece un laboratorio pequeño, no una sustitución de la ruta principal.

## Encaje estimado

| Escenario | Encaje aproximado |
|---|---:|
| 5+ años Full Stack con Java/Spring, React e inglés B2+ | 85-95 % |
| Java/Spring senior, frontend fuerte en Angular y React practicado | 65-80 % |
| Java sólido, React principalmente de proyectos y buen inglés | 58-72 % |
| Sin cinco años Full Stack o sin entrevista técnica en inglés | Riesgo alto |

## Requisitos clasificados

### Filtros principales

- Formación técnica/universitaria afín.
- Cinco años como Full Stack.
- Java y Spring Boot.
- React.
- Inglés intermedio/avanzado.
- Base de datos.
- Git, Postman y Confluence.
- Metodologías ágiles.

### Deseables

- Experiencia en seguros, banca, operaciones o telecomunicaciones.
- Cloud.

## Ambigüedades del anuncio

- “Java 8 a 25” puede significar experiencia con cualquier versión dentro del rango, convivencia de varios sistemas o capacidad de migración.
- No indica versión de Spring Boot.
- No identifica base de datos.
- No especifica proveedor ni servicios cloud.
- No describe responsabilidades concretas ni proporción backend/frontend.
- No indica frecuencia presencial.

Estas preguntas afectan mucho más la preparación que memorizar novedades de todas las versiones de Java.

## Java 8 a 25

No debes estudiar dieciocho versiones por separado. Organizar el repaso por saltos relevantes:

### Base Java 8

- lambdas e interfaces funcionales;
- Streams;
- Optional;
- `java.time`;
- `CompletableFuture`;
- colecciones y concurrencia.

### Java 9-11

- módulos a nivel conceptual;
- APIs de colecciones y strings;
- HTTP Client;
- cambios de runtime y dependencias removidas;
- migración de aplicaciones Java 8.

### Java 12-17

- switch expressions;
- text blocks;
- records;
- pattern matching inicial;
- sealed classes;
- impacto de Java 17 como baseline moderno.

### Java 18-21

- pattern matching maduro;
- virtual threads;
- structured concurrency a nivel de evolución;
- implicaciones para aplicaciones web y librerías.

### Java 22-25

- reconocer qué características son finales y cuáles preview;
- revisar compatibilidad de frameworks y build tools;
- no usar una feature moderna si producción compila con una versión anterior.

Preparar especialmente migración, compatibilidad y elección de runtime. No afirmar experiencia profesional en cada versión.

Material: [[JAVA]], [[Java Intermedio]], [[Java Avanzado]], [[08 - Virtual Threads]], [[04 - JVM Memoria y GC]].

## Spring Boot

La versión de Spring está condicionada por Java. Debes preguntar el par exacto runtime/framework.

Preparar:

- IoC/DI y configuración;
- REST controllers, DTOs y validación;
- manejo de errores;
- Spring Data y transacciones;
- seguridad;
- clientes HTTP y resiliencia;
- JUnit/Mockito/integration tests;
- Actuator, health y observabilidad;
- profiles, secretos y despliegue;
- compatibilidad al migrar Spring Boot 2 → 3/4.

Las versiones actuales de Spring Boot requieren como mínimo Java 17, mientras que líneas antiguas como 2.7 admiten Java 8-11. No se puede deducir la versión del proyecto solo por el rango del anuncio: [requisitos de Spring Boot](https://docs.spring.io/spring-boot/system-requirements.html).

Material: [[Spring Boot]], [[Spring Data]], [[Spring Security]], [[Testing en Java]], [[Performance Java]].

## React — laboratorio por segunda aparición

React ya aparece en dos vacantes, por lo que corresponde un laboratorio pequeño.

### Fundamentos mínimos

- componentes funcionales;
- props y composición;
- estado;
- hooks y reglas de hooks;
- efectos y cleanup;
- formularios y validación;
- routing;
- consumo de APIs;
- auth y rutas protegidas;
- manejo de carga/error/vacío;
- pruebas de componentes;
- performance básica y accesibilidad.

### Diferencias que debes manejar desde Angular

| Angular | React |
|---|---|
| framework completo | biblioteca de UI con ecosistema |
| DI y servicios integrados | composición, hooks y módulos propios |
| templates separados | JSX/TSX |
| RxJS frecuente | estado/effects y librerías opcionales |
| routing/forms oficiales o comunes | elección de librerías del proyecto |

No intentes escribir React como Angular. Aprovecha los fundamentos transferibles: TypeScript, componentes, HTTP, estado, testing y arquitectura por features.

## Integración Java + React

Preparar una feature end to end:

```text
React
  -> formulario y validación
  -> autenticación
  -> API Spring Boot
  -> caso de uso
  -> base de datos
  -> respuesta/errores
  -> UI y observabilidad
```

Temas:

- contrato OpenAPI;
- CORS;
- tokens y cookies según arquitectura;
- paginación, filtros y orden;
- manejo consistente de errores;
- versionado;
- cancelación y solicitudes duplicadas;
- pruebas de contrato.

## Base de datos

Como el anuncio no indica producto, preparar fundamentos transferibles:

- modelado relacional;
- SQL, joins, CTE y funciones de ventana;
- índices y planes de ejecución;
- transacciones, locks y aislamiento;
- migrations;
- ORM/JPA y consultas explícitas;
- connection pooling;
- paginación;
- diagnóstico de queries lentas;
- NoSQL solo si el proyecto lo confirma.

Preguntar producto, volumen, uso de procedures y responsabilidad del desarrollador.

Material: [[Bases de Datos Java]], [[01 - SQL para Backend Java]], [[03 - Indices y Queries Lentas]].

## Inglés técnico — filtro real

Preparar en inglés:

- presentación profesional de 90 segundos;
- explicación de arquitectura de un proyecto;
- historia de incidente y resolución;
- decisión técnica y trade-offs;
- explicación de una API Java/React;
- preguntas sobre equipo, proyecto y expectativas;
- vocabulario de reuniones, blockers, estimates y releases.

La meta no es sonar nativo. Debes entender preguntas, pedir aclaración, estructurar respuestas y comunicar decisiones sin cambiar constantemente al español.

Ensayar:

1. `Tell me about yourself.`
2. `Describe a challenging production issue.`
3. `Why did you choose this architecture?`
4. `How do you review code and handle disagreements?`
5. `Explain how the frontend integrates with your API.`

## Git, Postman y Confluence

### Git

- branching;
- pull requests;
- rebase/merge según política;
- resolución de conflictos;
- tags y releases;
- commits claros;
- revert seguro.

### Postman

- collections y environments;
- variables sin secretos versionados;
- auth;
- scripts y assertions;
- casos positivos/negativos;
- ejecución automatizada.

### Confluence

- documentación de APIs y flujos;
- ADRs;
- runbooks;
- notas de incidentes;
- páginas mantenibles y enlazadas;
- decisiones con fecha, contexto y responsables.

## Cloud

Es deseable y no tiene proveedor. Preparar conceptos:

- contenedores;
- compute administrado;
- base administrada;
- object storage;
- secretos e IAM;
- logging, métricas y trazas;
- autoscaling;
- CI/CD;
- alta disponibilidad y recuperación.

Usar Azure o AWS como evidencia propia; no estudiar un tercer proveedor sin confirmación.

## Sector empresarial

Seguros, banca, operaciones y telecom comparten necesidades frecuentes:

- auditoría y trazabilidad;
- seguridad y datos sensibles;
- resiliencia;
- integraciones legacy;
- transacciones e idempotencia;
- alta concurrencia;
- procesamiento batch/eventos;
- soporte y control de cambios.

Preparar experiencia real transferible. El sector es deseable, no excluyente.

## Proyecto demostrable

Crear una aplicación Java + React pequeña:

- Spring Boot REST API;
- React + TypeScript;
- base relacional;
- auth;
- filtros y paginación;
- validación y errores consistentes;
- pruebas backend/frontend;
- OpenAPI y Postman collection;
- Docker;
- despliegue en un cloud conocido;
- documentación de arquitectura en formato Confluence/Markdown;
- README en inglés.

## Preparación de siete días

| Día | Foco | Entregable |
|---|---|---|
| 1 | CV e inglés | timeline de cinco años y presentación en inglés |
| 2 | Java 8-25 | matriz de evolución y código compatible |
| 3 | Spring Boot | API, datos, seguridad y pruebas |
| 4 | React | feature completa integrada con API |
| 5 | Base de datos | modelo, índices y consulta optimizada |
| 6 | Herramientas/cloud | Postman, Git, documentación y deploy |
| 7 | Entrevista | system design y simulación completa en inglés |

Postular antes de completar el plan.

## Preguntas probables

- ¿Qué diferencias importantes existen entre Java 8, 11, 17, 21 y 25?
- ¿Cómo migrarías una aplicación Java 8?
- ¿Qué versión de Spring Boot usarías y por qué?
- ¿Cómo gestionas estado y efectos en React?
- ¿Cómo estructuras una aplicación React grande?
- ¿Cómo proteges la integración frontend/backend?
- ¿Cómo optimizas una consulta?
- ¿Cómo documentas una decisión técnica?
- Describe tu experiencia Full Stack durante los últimos cinco años.

Preparar las respuestas principales también en inglés.

## Palabras clave para el CV

`Java`, `Java 8`, `Java 11`, `Java 17`, `Java 21`, `Spring Boot`, `React`, `TypeScript`, `REST APIs`, `SQL`, `Git`, `Postman`, `Confluence`, `Agile`, `Cloud`, `Full Stack Development`, `English B2`.

No listar versiones de Java sin experiencia o conocimiento defendible.

## Preguntas para reclutamiento

1. ¿Los cinco años Full Stack son excluyentes?
2. ¿Qué porcentaje del trabajo corresponde a Java y React?
3. ¿Qué versión de Java usan actualmente?
4. ¿Qué versión de Spring Boot utilizan?
5. ¿Existe una migración entre versiones de Java?
6. ¿Qué base de datos utiliza el proyecto?
7. ¿Qué proveedor y servicios cloud emplean?
8. ¿La entrevista técnica se realiza en inglés?
9. ¿Qué nivel de uso diario del inglés requiere el puesto?
10. ¿A qué sector pertenece el cliente?
11. ¿Cuántos días presenciales requiere San Isidro?

## Decisión final

Es una vacante económicamente atractiva y una vía válida si puedes demostrar seniority Full Stack e inglés. Java/Spring queda aún más confirmado en el radar y React alcanza el umbral para un laboratorio.

No desplaza Angular/.NET; abre una segunda combinación concreta: **Java + React**.
