---
tags:
  - Postulaciones
  - GuíaMaestra
  - Entrevistas
Creado: 2026-08-30
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - 16 - Radar acumulativo de vacantes
  - 06 - Ruta Java Spring Boot como segundo eje backend
---
# Guía Maestra — Mi recomendación de preparación

No es un módulo más de la ruta. Es mi lectura personal de todo lo que hay registrado en [[POSTULACIONES]] — las 14 vacantes, el radar de frecuencias y las preguntas reales que ya te compartieron de un postulante similar — convertida en una sola decisión: **por dónde empezar, qué estudiar y qué te van a preguntar.**

Si solo vas a leer una nota de toda la carpeta, que sea esta.

## Mi recomendación en un párrafo

Empieza por consolidar evidencia de **.NET**, porque es el stack donde ya tienes trabajo real (no teoría) y es el que puedes demostrar hoy sin estudiar nada nuevo — eso te da postulaciones y entrevistas desde la semana 1. En paralelo, invierte el tiempo de estudio de las próximas 3-4 semanas en **Java/Spring**, no porque sea más importante que .NET, sino porque es donde tienes la brecha más grande frente a la demanda real (7 de 14 vacantes) y donde un candidato políglota se distingue del resto: la mayoría de quienes postulan a estas vacantes saben *un* backend, no dos. Azure, Docker, testing y seguridad no son una fase aparte: son el fondo común que defiendes en cualquier entrevista, así que se entrenan todas las semanas, no al final.

No esperes a "estar listo". La regla del 70% de [[00 - Empezar a postular mañana]] es correcta: postula ya con lo que tienes en .NET, y usa cada entrevista real como termómetro de qué brecha de Java o cloud te está deteniendo de verdad, en vez de adivinar.

## Orden de estudio que yo seguiría

### Fase 1 (semana 1) — Consolidar lo que ya sabes, no aprender nada nuevo

El objetivo no es estudiar, es poder **demostrar** lo que ya hiciste en .NET con evidencia concreta (repos, PRs, una app desplegada). Al mismo tiempo, empiezas a postular.

- Completar [[01 - Diagnóstico y matriz de competencias]] con honestidad — sin esto, todo lo demás se estudia a ciegas.
- Preparar CV maestro y presentación de 45 segundos con [[00 - Empezar a postular mañana]].
- Enviar postulaciones desde el día 1, priorizando las de "Prioridad alta/máxima" del radar: **Entelgy, TSOFT, Stefanini** — son las de mayor coincidencia directa con lo que ya sabes.

**Por qué primero y no Java:** una entrevista que consigues esta semana con lo que ya sabes vale más que dos semanas de estudio especulativo. Además, cada entrevista real te dice qué brecha te está bloqueando de verdad, no la que tú supones.

### Fase 2 (semanas 2-4) — Java/Spring al mismo nivel que .NET

Este es el bloque que más cambia tu situación, porque es donde está la brecha real y la demanda no cubierta. Sigue [[06 - Ruta Java Spring Boot como segundo eje backend]] día por día, pero no lo hagas aislado de .NET: cada concepto que aprendas en Java, anótalo contra su equivalente .NET (la tabla de esa nota es justo para eso). Aprender los dos a la vez, comparando, fija mejor que aprender Java "desde cero" como si .NET no existiera.

- Semana 2: Java moderno + Spring Core + Spring Boot/REST (días 1-2 de la ruta 17).
- Semana 3: Spring Data JPA + bases de datos + seguridad (días 3-4).
- Semana 4: testing + mensajería + microservicios (día 5) + primera simulación de entrevista en Java.

En paralelo, sigue postulando y ahora suma las vacantes Java-first: **ISP Consulting, Full Stack Senior Java Angular AWS, Java Senior BIAN, Full Stack Java React**.

### Fase 3 (semanas 5-6) — El fondo común que decide entrevistas: cloud, contenedores, arquitectura

No es que Azure/Docker/patrones sean menos importantes — es que sin backend sólido no hay contexto para aplicarlos. Ahora sí conviene profundizar:

- Docker multi-stage + Compose + CI/CD (nota 07).
- Azure: App Service, Key Vault, Application Insights (nota 06) — con vocabulario transferible a AWS/GCP para las vacantes que los piden.
- Arquitectura: capas, hexagonal, SAGA, Cache-Aside — ya documentados en `05_🏗️ Arquitectura/ARQUITECTURA.md`. Estos temas son los que separan una entrevista media de una senior.

### Fase 4 (semanas 7-8) — Simulacros y cierre de portafolio

- Dos simulaciones técnicas completas (una en .NET, una en Java) y una conductual.
- Cerrar el proyecto de portafolio con README, ADR y demo — [[11 - Proyecto de portafolio demostrable]].
- Revisar métricas reales de postulaciones y ajustar: si hay screenings pero no técnicas, el problema es la narrativa, no el estudio.

> [!note] Esto no reemplaza [[07 - Plan de ocho semanas]] ni [[03 - Horario Diario Realista (8am-8pm)]]
> La nota 14 tiene el detalle operativo semana a semana (horas, rutina, checklist de domingo); la 18 lo baja a horario de reloj para una jornada 8am-8pm. Esta guía es la decisión de fondo: **qué va primero y por qué**. Las tres se usan juntas.

## Qué te van a preguntar — por categoría, de mayor a menor probabilidad

Basado en las preguntas reales que ya te compartieron de un postulante similar, más lo que exige el conjunto de las 14 vacantes.

### 1. Arquitectura y patrones (la categoría que más se repite en las entrevistas reales que ya viste)

Esto es lo que separa a un desarrollador de un ingeniero en la entrevista. No basta con definir el patrón — te van a pedir un caso donde lo aplicaste o dónde NO lo usarías.

- **SAGA**: ¿coreografía u orquestación? ¿Cómo manejas la compensación si falla el paso 3 de 5?
- **Arquitectura hexagonal**: ¿qué es un puerto y qué es un adaptador? ¿Por qué el dominio no debería depender de Spring/JPA?
- **Arquitectura en capas**: ¿cuál es la regla de dependencia? ¿Qué es un modelo de dominio anémico y por qué es un problema?
- **Cache-Aside**: ¿qué pasa si el caché y la base de datos quedan inconsistentes?
- **Patrones de diseño (GoF)**: Circuit Breaker (¿qué estados tiene: closed/open/half-open?), Factory Method vs Abstract Factory, Singleton (¿por qué el enum singleton es el más seguro en Java?), Repository (¿qué problema resuelve frente a acceder a JPA directo?), Strategy (¿cómo evitas un `if/else` gigante?).
- **DDD/CQRS**: si una vacante lo menciona (Backend Java NET DDD DB2), prepara un caso aplicado, no la definición de libro.

Repasar: `06_Patrones/` en el vault de Java, y las secciones de arquitectura ya documentadas en `ARQUITECTURA.md`.

### 2. Backend específico — Java o .NET según la vacante

- **Java/Spring**: Lombok (¿qué genera y qué riesgo tiene con entidades JPA?), JPA (N+1, `FetchType`, transacciones), Spring Boot (inyección de dependencias, `@Transactional`, manejo de errores), Kafka (particiones, consumer groups, exactly-once), Redis (cache-aside, TTL, cuándo NO usarlo).
- **Quarkus**: por qué build-time processing en vez de runtime, cuándo lo elegirías sobre Spring Boot (arranque rápido, GraalVM, Cloud Native).
- **.NET**: lifetime de `DbContext`, qué pasa internamente en `async/await`, `AsNoTracking`, `ProblemDetails`, idempotencia de un endpoint.

Ver el banco completo por stack en [[15 - Banco de preguntas Full Stack]] (incluye ahora bloque Java y bloque .NET).

### 3. Bases de datos

- Índices: ¿cómo verificas que un índice se está usando realmente?
- ACID vs BASE, CAP theorem — cuándo aceptar consistencia eventual.
- SQL vs NoSQL: no "cuál es mejor" sino "cuándo un documento gana a una tabla".
- Oracle/PL-SQL si la vacante lo pide (TSOFT, Java Senior BIAN): procedures, functions, triggers.

### 4. Cloud y DevOps

- Diferencia entre PaaS, IaaS y Cloud Native.
- Docker multi-stage, health checks, por qué liveness y readiness no son lo mismo.
- CI/CD: cómo estructurarías build → test → image → deploy, y cómo harías rollback.
- Mensajería: cómo evitas procesar un mensaje dos veces (idempotencia + Outbox).

### 5. Seguridad

- OWASP Top 10 — que puedas nombrar y explicar al menos 6 de los 10 con un ejemplo concreto (ya está mapeado en `03 - Seguridad Backend Java.md`).
- Autorización por recurso, no solo por rol.
- Qué nunca debe aparecer en un log (tokens, passwords).

### 6. Conductuales / fit

Estas preguntas deciden tanto como las técnicas y se preparan menos de lo debido:

- Cuéntame un error técnico que cometiste y qué aprendiste.
- Describe un desacuerdo de arquitectura con otro desarrollador — ¿cómo se resolvió?
- ¿Cómo priorizas deuda técnica frente a una entrega con fecha fija?
- Cuéntame un incidente de producción difícil: contexto, decisión, resultado.
- ¿Qué harías en tus primeros 30 días en el equipo?

Prepáralas con la fórmula **situación → acción → resultado**, con datos reales, nunca inventados. Ver [[13 - Sistema de postulación por vacante]] para la fórmula de logros aplicada al CV.

## Cómo prepararte, no solo qué

1. **Nunca respondas solo con la definición.** Definición + ejemplo propio + trade-off es el patrón que se repite en todas las preguntas de arriba. Practica en voz alta, no solo mentalmente.
2. **Ten un caso real por patrón**, no diez patrones memorizados sin ejemplo. Es preferible defender bien 5 patrones con caso real que listar 15 de memoria.
3. **Diferencia explícitamente Java de .NET cuando te pregunten** — usa la tabla de equivalencias de [[06 - Ruta Java Spring Boot como segundo eje backend]] para no mezclar vocabulario a mitad de una respuesta.
4. **Antes de cada entrevista técnica**, repasa la nota puntual de esa vacante (creada con la plantilla de [[13 - Sistema de postulación por vacante]]) para saber qué requisitos son obligatorios vs deseables en esa oferta específica.
5. **Simula la entrevista completa**, no solo preguntas sueltas: 45 segundos de pitch, una pregunta de arquitectura, una de código, una conductual, en ese orden — así se sienten la mayoría de las entrevistas reales.

## Errores que evitaría en tu lugar

- Estudiar Kubernetes/AKS a fondo antes de tener Docker y CI/CD sólidos — es la trampa más común y ninguna de las 14 vacantes lo exige como núcleo.
- Presentarte siempre como ".NET" cuando la vacante pide Java, o viceversa — usa el discurso correcto de la sección "Kit universal" en [[RUTA FULL STACK ANGULAR NET AZURE]].
- Memorizar la lista de OWASP Top 10 sin poder mapear cada uno a un ejemplo de código — en entrevista siempre piden el ejemplo.
- Dejar de postular mientras "te preparas" — las entrevistas reales son la señal más confiable de qué estudiar después, mejor que cualquier plan teórico.
- Tratar Redis/Kafka/mensajería como opcional — aparecen en 4 de 14 vacantes y casi siempre se pregunta por idempotencia, que es fácil de responder bien si lo preparaste una vez.

## Checklist final antes de cualquier entrevista

- [ ] Sé si esta vacante es .NET-first, Java-first o políglota, y elegí el discurso correcto.
- [ ] Repasé los requisitos obligatorios de esa vacante puntual, no solo la ruta general.
- [ ] Tengo al menos un caso real preparado para arquitectura (capas, hexagonal, SAGA o Cache-Aside).
- [ ] Tengo al menos un caso real preparado para un patrón de diseño (Circuit Breaker, Strategy, Repository).
- [ ] Puedo explicar idempotencia y por qué importa en mensajería.
- [ ] Tengo dos historias conductuales listas (un error, un incidente) en formato situación-acción-resultado.
- [ ] Sé qué le voy a preguntar yo a ellos al final.
