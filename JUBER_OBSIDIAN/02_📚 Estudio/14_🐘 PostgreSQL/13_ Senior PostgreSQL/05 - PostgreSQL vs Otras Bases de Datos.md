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

# 05 - PostgreSQL vs Otras Bases de Datos

## Que vas a aprender

En este capitulo vas a cerrar el capitulo consolidando una comparativa objetiva de Postgres frente a otros motores relevantes: **MySQL**, **SQL Server** (que estudiaras en el libro complementario), y bases de datos **NoSQL** (MongoDB, Redis), con criterios reales de decision.

## Explicacion

Frente a **MySQL**, Postgres generalmente ofrece mayor riqueza de tipos de datos (JSONB, arrays, ver capitulo de Tipos Avanzados) y mayor adherencia estricta al estandar SQL, mientras que MySQL historicamente tuvo ventajas de simplicidad operativa y un ecosistema de hosting muy amplio (aunque esta diferencia se redujo significativamente en los ultimos años). Frente a **SQL Server** (el motor de tu otro libro complementario, usado en StaffTrainingPlatform), la eleccion suele estar mas determinada por el ecosistema (Postgres domina en Node/NestJS, SQL Server es natural en .NET) que por diferencias tecnicas fundamentales, dado que ambos son motores relacionales maduros y capaces.

Frente a bases de datos **NoSQL**: MongoDB (documental) tiene sentido cuando el modelo de datos es genuinamente sin esquema fijo y no se necesitan transacciones complejas entre entidades relacionadas (aunque Postgres con JSONB, ver capitulo de Tipos Avanzados, cubre buena parte de esos casos de uso sin sacrificar transacciones ACID). Redis (ya visto extensamente en el libro de NestJS) no compite directamente con Postgres: es una base de datos en memoria para cache, sesiones, y colas, complementaria a una base de datos relacional, no un reemplazo.

## Modelo mental

Pensa la eleccion entre motores de base de datos como elegir el vehiculo correcto segun el terreno real que vas a recorrer, no segun cual vehiculo es "objetivamente mejor" en una revista especializada: Postgres es un vehiculo todoterreno muy capaz que sirve para la gran mayoria de caminos (transaccional, analitico moderado, datos semi-estructurados), mientras que herramientas mas especializadas (Redis para velocidad extrema en memoria, MongoDB para esquemas genuinamente variables) tienen sentido solo cuando el terreno especifico realmente lo demanda.

## Ejemplo base

```
Comparativa conceptual simplificada:

| Motor       | Fortaleza principal                          | Cuando elegirlo                              |
|--------------|-----------------------------------------------|------------------------------------------------|
| PostgreSQL   | Riqueza de tipos, extensibilidad, ACID solido | Default razonable para la mayoria de proyectos |
| MySQL        | Simplicidad operativa, ecosistema hosting     | Legacy, o equipos ya expertos en el            |
| SQL Server   | Integracion profunda con .NET                 | Ecosistema Microsoft/.NET (ver libro dedicado) |
| MongoDB      | Esquema genuinamente variable, escala horizontal| Datos sin estructura relacional real          |
| Redis        | Velocidad en memoria, estructuras simples      | Cache, sesiones, colas (complementario, no reemplazo) |
```

## Como se usa en un proyecto real

1. Se usa Postgres como default razonable para la gran mayoria de proyectos nuevos, salvo un requisito especifico que apunte claramente a otra opcion.
2. Se elige SQL Server cuando el ecosistema ya es .NET (ver libro complementario y el caso de StaffTrainingPlatform), priorizando integracion sobre diferencias tecnicas marginales.
3. Se evalua NoSQL (MongoDB) solo cuando el modelo de datos genuinamente no se beneficia de estructura relacional, no como eleccion por moda.
4. Se usa Redis siempre como complemento de una base de datos relacional (cache, sesiones, colas, ver libro de NestJS), nunca como reemplazo de Postgres para datos que necesitan transacciones e integridad referencial real.
5. Se documenta la eleccion de motor como una decision arquitectonica con el mismo rigor ya aplicado en todo este ecosistema de libros, evitando decisiones por preferencia personal sin justificacion real.

## Criterio junior

Un junior debe conocer las diferencias basicas entre Postgres, MySQL, SQL Server, y por que Redis no compite directamente con ellos.

## Criterio mid

Un mid debe justificar la eleccion de motor para un proyecto especifico segun el ecosistema y los requisitos reales de datos.

## Criterio senior

Un senior debe liderar decisiones de eleccion de motor de base de datos para sistemas completos, documentando esa decision como un ADR con criterios objetivos, no preferencia personal.

## Errores comunes

- Elegir un motor NoSQL por moda cuando el modelo de datos real se beneficiaria claramente de estructura relacional con integridad referencial.
- Usar Redis como reemplazo de una base de datos relacional para datos que necesitan transacciones ACID reales, en vez de como complemento.
- No considerar el ecosistema del equipo (Node/NestJS vs .NET) como factor real de decision, evaluando solo diferencias tecnicas abstractas.
- No documentar la eleccion de motor como decision arquitectonica, dejandola como una preferencia personal no justificada.

## Practica

- [ ] Compara Postgres y MySQL para un caso hipotetico, considerando tipos de datos y ecosistema.
- [ ] Compara Postgres y SQL Server para un caso hipotetico, considerando el ecosistema de aplicacion (Node vs .NET).
- [ ] Identifica un caso real donde MongoDB seria mas apropiado que Postgres, y explica por que.
- [ ] Explica por que Redis no reemplaza a Postgres, sino que lo complementa.
- [ ] Documenta la eleccion de motor de un proyecto real tuyo como un ADR con criterios objetivos.

## Preguntas de repaso

- Que ventaja tipica ofrece Postgres frente a MySQL en cuanto a tipos de datos y adherencia al estandar?
- Que factor suele determinar mas la eleccion entre Postgres y SQL Server que diferencias tecnicas fundamentales?
- Cuando se justificaria realmente elegir MongoDB sobre Postgres con JSONB?
- Por que Redis nunca deberia considerarse un reemplazo de una base de datos relacional para datos transaccionales?

## Cierre del libro

Completar este capitulo cierra el recorrido completo de PostgreSQL desde fundamentos hasta criterio senior: SQL solido, modelado relacional correcto, indices y optimizacion basados en evidencia, transacciones y MVCC, programabilidad, tipos avanzados, administracion y alta disponibilidad, despliegue en la nube, integracion profunda con NestJS/TypeORM, performance avanzada, y ahora criterio senior de arquitectura, seguridad y comparacion objetiva con otros motores. El proximo paso natural es aplicar este conocimiento en tus proyectos reales, y compararlo con lo que aprenderas en el libro complementario de [[SQL SERVER]], especialmente relevante para StaffTrainingPlatform.
