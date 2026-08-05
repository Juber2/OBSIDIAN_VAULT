---
tags:
  - NestJS
  - TypeScript
  - Nest-Senior
Creado: 2026-07-27
Relacionado:
  - Senior NestJS
  - NESTJS
---

# 05 - Liderazgo Tecnico Backend

## Que vas a aprender

En este capitulo vas a desarrollar las habilidades de **liderazgo tecnico** necesarias para un rol senior en un equipo backend NestJS: mentoria, definicion de estandares de equipo, y como equilibrar entrega de features con salud tecnica a largo plazo.

## Explicacion

El liderazgo tecnico no es un titulo, es una funcion: alguien que ayuda al equipo a tomar mejores decisiones colectivas, no solo a escribir mejor codigo individualmente. Esto incluye mentoria activa (no solo corregir en code review, sino explicar el razonamiento para que la persona aprenda el criterio, no solo el fix puntual), definicion de estandares compartidos (las convenciones vistas en cada capitulo de este libro: capas, testing, seguridad, arquitectura) documentados y accesibles, y la habilidad de negociar con producto/negocio cuando la presion de entrega choca con la necesidad de mantener calidad tecnica.

Un lider tecnico efectivo tambien sabe delegar decisiones, no centralizar todo: el objetivo es que el equipo tome buenas decisiones incluso sin el lider presente, lo cual requiere invertir en que el criterio (no solo las reglas) se transmita.

## Modelo mental

Pensa el liderazgo tecnico como ser el jardinero de un sistema vivo, no el unico que construye cada parte: el trabajo es crear las condiciones (estandares claros, mentoria, procesos) para que el equipo completo crezca y produzca buen codigo de forma sostenida, no ser el heroe que resuelve todo personalmente.

## Ejemplo base

```
Practicas concretas de liderazgo tecnico backend:

- Pair programming ocasional en features complejas, no solo review despues del hecho.
- Documentar el "por que" de decisiones (ADRs) para que el criterio se transmita, no solo la regla.
- Rotar quien revisa PRs criticos, para que el conocimiento no quede concentrado en una persona.
- Presupuestar tiempo explicito para deuda tecnica en cada sprint/ciclo, no como "cuando sobre tiempo".
- Negociar con producto mostrando el costo real (en tiempo futuro) de no atender riesgos tecnicos identificados.
```

## Como se usa en un proyecto real

1. Se invierte tiempo real en mentoria, no solo en corregir codigo: explicar el razonamiento detras de cada sugerencia de review.
2. Se documentan y socializan los estandares del equipo (basados en los capitulos de este libro) en un lugar accesible, no solo en la cabeza del lider.
3. Se presupuesta tiempo explicito para atender deuda tecnica y refactors, negociado como parte normal de la planificacion, no como excepcion.
4. Se rota la responsabilidad de decisiones y reviews criticos entre el equipo, evitando un unico punto de fallo de conocimiento.
5. Se mide y comunica el impacto de decisiones tecnicas en terminos que negocio entiende (tiempo de desarrollo futuro, incidentes evitados), no solo en jerga tecnica.

## Criterio junior

Un junior en camino a mid debe poder recibir mentoria activamente, aplicar el feedback entendiendo el razonamiento, y empezar a mentorear a quien recien se suma con temas que ya domina.

## Criterio mid

Un mid en camino a senior debe poder liderar decisiones tecnicas de alcance moderado, dar mentoria a otros de forma consistente, y contribuir a definir estandares de equipo con argumentos, no solo preferencias.

## Criterio senior

Un senior debe liderar la salud tecnica del equipo a largo plazo: balancear entrega con calidad, formar a la proxima generacion de seniors, y ser la voz que negocia con claridad el costo tecnico de decisiones de negocio apresuradas.

## Errores comunes

- Concentrar todo el conocimiento critico en una sola persona, generando un punto unico de fallo para el equipo.
- Dar feedback en review sin explicar el razonamiento, perdiendo la oportunidad de que la persona aprenda el criterio general.
- Nunca presupuestar tiempo para deuda tecnica, dejandola acumularse hasta que se vuelve un problema critico e inevitable.
- Ceder siempre a la presion de entrega sin comunicar el costo tecnico real de esas decisiones a quienes deciden prioridades.

## Practica

- [ ] Da feedback en un review real explicando el razonamiento completo, no solo el fix puntual.
- [ ] Documenta un estandar de equipo (de cualquier capitulo de este libro) en un lugar accesible para todos.
- [ ] Propone presupuestar tiempo explicito para deuda tecnica en la proxima planificacion.
- [ ] Identifica un punto unico de fallo de conocimiento en tu equipo actual y proponé como distribuirlo.
- [ ] Anota como comunicarias, en terminos de negocio, el costo de no atender un riesgo tecnico identificado.

## Preguntas de repaso

- Que diferencia hay entre corregir un bug en review y mentorear a traves de ese mismo review?
- Por que concentrar conocimiento critico en una sola persona es un riesgo para el equipo?
- Como negociarias con producto el tiempo necesario para atender deuda tecnica importante?
- Que significa que el liderazgo tecnico "cree las condiciones" en vez de resolver todo personalmente?
