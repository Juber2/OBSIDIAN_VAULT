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

# 04 - Code Review de Queries SQL

## Que vas a aprender

En este capitulo vas a consolidar una checklist de **code review especifica para SQL y migraciones**, aplicando el mismo criterio de revision senior ya visto en el capitulo de Code Review Senior del libro de NestJS, ahora enfocado en queries y esquema de base de datos.

## Explicacion

Un code review senior de codigo relacionado con base de datos revisa capas que un review general de aplicacion podria pasar por alto: si una query nueva tiene el indice apropiado (ver capitulo de Indices) verificable con `EXPLAIN`, si una migracion en una tabla grande usa tecnicas zero-downtime (tema anterior) cuando corresponde, si constraints nuevos (ver capitulo de Modelado) replican reglas de negocio criticas como defensa en profundidad, y si SQL crudo (ver capitulo de Postgres con NestJS) esta correctamente parametrizado sin riesgo de inyeccion.

Automatizar lo automatizable libera al reviewer humano para enfocarse en decisiones de diseño: un linter de SQL o una revision automatica de que toda migration tenga un `down` funcional puede ser parte del pipeline de CI (ver libro de Docker), mientras que el review humano se concentra en si el modelado de datos es correcto, si una query nueva escala con el volumen esperado, y si una decision de indexacion o particionamiento tiene sentido para el patron de uso real.

## Modelo mental

Pensa el code review de SQL como una inspeccion de ingenieria civil antes de aprobar los planos de una ampliacion de edificio: no solo revisa que la sintaxis del plano sea correcta (eso lo puede verificar una herramienta automatica), sino que la estructura realmente soporte el peso esperado (indices para el volumen de datos), que no debilite los cimientos existentes (constraints y integridad referencial), y que la construccion pueda hacerse sin tener que evacuar el edificio completo (zero-downtime).

## Ejemplo base

```
Checklist minima de review para SQL/migraciones de Postgres:

[ ] Cualquier query nueva sobre una tabla grande tiene el indice apropiado (verificado con EXPLAIN).
[ ] Migraciones en tablas grandes usan CREATE INDEX CONCURRENTLY y NOT VALID + VALIDATE.
[ ] Cada migration tiene un down funcional, probado localmente.
[ ] Constraints replican reglas de negocio criticas ya validadas en la aplicacion.
[ ] SQL crudo esta parametrizado, sin concatenacion de valores de usuario.
[ ] Cambios de esquema documentados si no son obvios (por que un indice GIN, por que un CHECK especifico).
[ ] Politicas de RLS (si aplica) testeadas para el nuevo caso.
```

## Como se usa en un proyecto real

1. Se usa una checklist compartida (basada en todos los capitulos de este libro) como base minima de cualquier review que toque SQL o esquema de base de datos.
2. Se automatiza lo automatizable (verificacion de `down` presente, linting de SQL basico) en el pipeline de CI, liberando al reviewer humano para decisiones de diseño mas complejas.
3. Se revisa con especial atencion cualquier migracion sobre una tabla identificada como grande o de alto trafico, aplicando las tecnicas zero-downtime del tema anterior cuando corresponda.
4. Se verifica con `EXPLAIN` (no solo se asume) que una query nueva efectivamente usa los indices esperados antes de aprobar el cambio.
5. Se da feedback explicando el razonamiento (aplicando el mismo principio de mentoria ya visto en el capitulo de Liderazgo Tecnico del libro de NestJS), no solo señalando el problema.

## Criterio junior

Un junior debe poder seguir una checklist de review dada y detectar problemas obvios (falta de `down`, SQL sin parametrizar).

## Criterio mid

Un mid debe hacer reviews proactivos verificando indices con `EXPLAIN` y tecnicas zero-downtime en migraciones de tablas grandes.

## Criterio senior

Un senior debe liderar la cultura de review de codigo relacionado con base de datos, definiendo la checklist compartida y mentoreando a traves del feedback en cada revision.

## Errores comunes

- Aprobar una migracion sin verificar si necesita tecnicas zero-downtime segun el tamaño real de la tabla afectada.
- No verificar con `EXPLAIN` que una query nueva realmente usa los indices esperados, aprobando basandose solo en que "se ve razonable".
- No revisar si SQL crudo esta correctamente parametrizado, dejando pasar un riesgo de inyeccion SQL.
- Dar feedback sin explicar el razonamiento, perdiendo la oportunidad de que el review tambien sea una instancia de aprendizaje para el equipo.

## Practica

- [ ] Define una checklist de review de SQL/migraciones para tu equipo, basada en los capitulos de este libro.
- [ ] Revisa una migration real (o simulada) aplicando esa checklist.
- [ ] Verifica con `EXPLAIN` si una query de un PR reciente usa los indices esperados.
- [ ] Identifica un caso donde SQL crudo no estaba correctamente parametrizado y corrigelo.
- [ ] Practica dar feedback explicando el "por que" en una revision simulada de SQL.

## Preguntas de repaso

- Que verificarias siempre en una migracion que afecta una tabla grande de produccion?
- Como confirmarias, mas alla de la intuicion, que una query nueva realmente usa el indice esperado?
- Que riesgo especifico de seguridad revisarias en cualquier SQL crudo dentro de un PR?
- Que deberia estar automatizado en CI en vez de depender de un reviewer humano recordarlo cada vez?
