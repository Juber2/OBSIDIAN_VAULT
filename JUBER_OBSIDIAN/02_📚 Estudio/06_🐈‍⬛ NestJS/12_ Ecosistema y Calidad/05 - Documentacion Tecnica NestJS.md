---
tags:
  - NestJS
  - TypeScript
  - Nest-Calidad
Creado: 2026-07-27
Relacionado:
  - Ecosistema y Calidad
  - NESTJS
---

# 05 - Documentacion Tecnica NestJS

## Que vas a aprender

En este capitulo vas a aprender a documentar un proyecto NestJS de forma que sea util a largo plazo: README efectivo, documentacion de arquitectura (ADRs), y como mantener la documentacion viva en vez de que se desactualice con el codigo.

## Explicacion

La documentacion mas valiosa no es la que describe "que hace el codigo" (eso deberia leerse del codigo mismo, con buenos nombres), sino la que explica **por que** se tomo una decision que no es obvia mirando solo el codigo actual. Un **ADR** (Architecture Decision Record) es un documento corto que registra una decision arquitectonica importante, el contexto que la motivo, las alternativas consideradas, y las consecuencias aceptadas; es invaluable meses despues cuando alguien se pregunta "por que esta hecho asi" y nadie recuerda la razon.

El README de un proyecto Nest deberia responder rapido las preguntas que cualquier nuevo integrante necesita: como levantar el proyecto localmente, que variables de entorno necesita, como correr tests, y un mapa general de la estructura de modulos.

## Modelo mental

Pensa la documentacion de codigo como explicar "que hace esta maquina" (el codigo ya lo dice, con buenos nombres), y un ADR como explicar "por que elegimos esta maquina y no otra" (eso el codigo nunca lo va a decir por si solo, sin importar que tan bien escrito este).

## Ejemplo base

```markdown
# ADR-003: Usar BullMQ para procesamiento de reportes

## Contexto
Los reportes tardan entre 30s y 5min en generarse. Generarlos de forma sincrona
bloqueaba el Event Loop y afectaba la latencia de otros endpoints.

## Decision
Mover la generacion de reportes a una cola BullMQ respaldada por Redis,
con un worker en un proceso separado.

## Alternativas consideradas
- Worker threads: descartado, complica el manejo de reintentos y persistencia.
- Microservicio dedicado: descartado por ahora, complejidad no justificada aun.

## Consecuencias
- Se suma Redis como dependencia de infraestructura.
- Los reportes ya no son inmediatos; el cliente debe consultar el estado del job.
```

## Como se usa en un proyecto real

1. Se mantiene un README con instrucciones de setup local siempre actualizadas, verificadas cada tanto por alguien que no conoce el proyecto de memoria.
2. Se escribe un ADR corto para cada decision arquitectonica no obvia (eleccion de ORM, estrategia de auth, adopcion de un patron como CQRS).
3. Los ADRs se guardan versionados en el repositorio (`docs/adr/`), nunca en un documento externo que se desincroniza del codigo.
4. Se evita documentar detalles que cambian frecuentemente (esos se desactualizan rapido); se documenta el "por que" que es mas estable que el "como" exacto.
5. La documentacion de API (Swagger, ya visto) se mantiene como fuente separada, generada del codigo, no duplicada manualmente en el README.

## Criterio junior

Un junior debe poder seguir un README para levantar el proyecto localmente sin ayuda adicional, y escribir un ADR simple siguiendo una plantilla.

## Criterio mid

Un mid debe identificar que decisiones merecen un ADR (no todas) y mantener el README sincronizado con cambios reales de setup del proyecto.

## Criterio senior

Un senior debe establecer la cultura de documentacion del equipo (que se documenta, donde, con que plantilla), revisar que los ADRs reflejen decisiones reales tomadas en conjunto, y usar ese historial para acelerar el onboarding de nuevos integrantes.

## Errores comunes

- Documentar el "que" (que ya es evidente leyendo el codigo) en vez del "por que" (que no lo es).
- Dejar el README desactualizado despues de cambios de setup, generando friccion para cualquier nuevo integrante.
- No registrar decisiones arquitectonicas importantes, perdiendo el contexto meses despues cuando alguien cuestiona por que algo esta hecho de cierta forma.
- Mantener documentacion de API duplicada manualmente en vez de generada automaticamente desde el codigo (Swagger).

## Practica

- [ ] Escribe o actualiza el README de un proyecto de practica con instrucciones de setup completas.
- [ ] Escribe un ADR para una decision arquitectonica real (o hipotetica) de tu proyecto.
- [ ] Verifica que el README siga siendo preciso siguiendolo paso a paso desde cero.
- [ ] Crea la carpeta `docs/adr/` en un proyecto y agrega una plantilla base de ADR.
- [ ] Anota que decisiones de tu proyecto actual merecerian un ADR retroactivo.

## Preguntas de repaso

- Que diferencia hay entre documentar el "que" y documentar el "por que"?
- Que informacion deberia responder un buen README de un proyecto Nest?
- Que partes de un ADR son las mas valiosas a largo plazo?
- Por que la documentacion de API deberia generarse del codigo en vez de mantenerse duplicada a mano?
