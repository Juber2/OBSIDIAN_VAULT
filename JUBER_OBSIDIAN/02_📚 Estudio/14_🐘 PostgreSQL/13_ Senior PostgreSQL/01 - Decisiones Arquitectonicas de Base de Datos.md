---
tags:
  - PostgreSQL
  - Senior
  - Postgres-Senior
Creado: 2026-07-29
Relacionado:
  - Senior PostgreSQL
  - POSTGRESQL
---

# 01 - Decisiones Arquitectonicas de Base de Datos

## Que vas a aprender

En este capitulo vas a consolidar el criterio senior para tomar decisiones arquitectonicas relacionadas con la base de datos, aplicando el mismo marco de decision ya usado consistentemente en los libros de Docker, NestJS y Linux, ahora especifico de Postgres.

## Explicacion

Decisiones como "base de datos unica versus una por servicio" (en una arquitectura de microservicios, ver capitulo de Arquitectura del libro de NestJS), "cuando particionar" (ver capitulo de Performance Avanzada), "cuando introducir replicas de lectura" (ver capitulo de Alta Disponibilidad), o "PostgreSQL versus otro motor" no tienen una respuesta universal correcta: dependen del contexto real del sistema, el equipo, y el negocio. El mismo marco de cinco preguntas ya aplicado en los otros libros de esta serie (que problema real resuelve, que pasa si no se hace, que complejidad introduce, es reversible, el equipo tiene experiencia) aplica exactamente igual aca.

Una decision especifica de bases de datos que merece atencion senior particular es **cuando dividir una base de datos monolitica** en multiples bases independientes (por servicio, o por dominio): esto elimina la posibilidad de hacer JOINs directos entre esos dominios (ver capitulo de Esquemas), forzando composicion a nivel de aplicacion, un tradeoff real que debe evaluarse cuidadosamente, no aplicarse por moda de microservicios.

## Modelo mental

Pensa cada decision arquitectonica de base de datos como elegir la estructura de un archivo compartido en una organizacion: un unico archivo central es simple de consultar pero se vuelve un cuello de botella y un punto unico de fallo a medida que la organizacion crece; dividir en archivos departamentales da autonomia pero complica cualquier consulta que necesite cruzar informacion de varios departamentos a la vez. Ninguna opcion es "la correcta" en abstracto, depende del tamaño real y las necesidades reales de esa organizacion especifica.

## Ejemplo base

```
Marco de decision aplicado a base de datos (mismo marco de toda la serie):

1. Que problema real resuelve esta decision (dividir bases, particionar, agregar replica), hoy?
2. Que pasa si NO se hace?
3. Que complejidad operativa y de desarrollo introduce (perdida de JOINs directos, mas infraestructura)?
4. Es reversible? Que tan caro seria deshacer esta decision en 6 meses?
5. El equipo tiene experiencia real para operar esto con confianza?
```

## Como se usa en un proyecto real

1. Se empieza con una unica base de datos Postgres bien modelada (con esquemas separados si es necesario, ver capitulo de Esquemas) como default razonable, evitando dividir prematuramente por moda de microservicios.
2. Se aplica el marco de cinco preguntas antes de cualquier decision significativa (particionar, replicar, dividir bases), documentandola como un ADR.
3. Se evalua division de bases de datos solo cuando equipos verdaderamente independientes necesitan autonomia de despliegue y esquema, aceptando el costo de composicion a nivel de aplicacion.
4. Se revisa periodicamente si decisiones tomadas anteriormente siguen siendo apropiadas a medida que el sistema y equipo crecen.
5. Se conecta esto explicitamente con el mismo marco de decision ya usado en Docker (Compose vs orquestador) y Linux (infraestructura), demostrando consistencia de criterio en todo este ecosistema de conocimiento.

## Criterio junior

Un junior debe poder seguir decisiones arquitectonicas de base de datos ya tomadas, entendiendo el razonamiento si esta documentado.

## Criterio mid

Un mid debe aplicar el marco de decision a un caso concreto, cuestionando constructivamente decisiones que no encajan con la realidad del proyecto.

## Criterio senior

Un senior debe liderar decisiones arquitectonicas de base de datos para sistemas completos, documentarlas como ADRs, y revisarlas periodicamente con el mismo rigor aplicado en toda esta serie de libros.

## Errores comunes

- Dividir una base de datos en multiples bases por moda de microservicios sin una necesidad real de autonomia de equipos independientes.
- No documentar el razonamiento detras de decisiones significativas de base de datos, perdiendo ese contexto con el tiempo.
- Aplicar un criterio distinto para decisiones de base de datos que el ya usado consistentemente para decisiones de infraestructura y arquitectura de aplicacion en el resto de esta serie.
- No revisar decisiones periodicamente, quedando atado a una estructura de base de datos que ya no encaja con la realidad actual del sistema.

## Practica

- [ ] Aplica el marco de cinco preguntas a una decision de base de datos real o hipotetica de tu contexto (particionar, dividir, replicar).
- [ ] Documenta esa decision como un ADR.
- [ ] Identifica una decision de base de datos de un proyecto que conozcas que ya no encaja con su realidad actual.
- [ ] Compara este marco con el ya usado en el libro de Docker para "Compose vs orquestador".
- [ ] Evalua si tu proyecto actual se beneficiaria de dividir su base de datos o si una unica base bien modelada sigue siendo apropiada.

## Preguntas de repaso

- Que tradeoff real introduce dividir una base de datos monolitica en multiples bases por dominio?
- Como se relaciona el marco de decision de este capitulo con el ya usado en el libro de Docker?
- Por que empezar con una unica base de datos bien modelada suele ser la decision correcta por defecto?
- Como te asegurarias de que una decision arquitectonica de base de datos se revise cuando el contexto cambie?
