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

# 03 - Paquetes NestJS Clave

## Que vas a aprender

En este capitulo vas a hacer un repaso curado del **ecosistema oficial y de terceros** de NestJS: paquetes que resuelven necesidades comunes, para no reinventar soluciones que ya existen y estan mantenidas.

## Explicacion

NestJS tiene un ecosistema oficial extenso (`@nestjs/*`) que cubre la mayoria de necesidades de una aplicacion backend real, ademas de integraciones bien establecidas con librerias populares del ecosistema Node. Conocer este mapa evita reinventar ruedas: antes de escribir una solucion custom para "validar variables de entorno" o "generar documentacion de API", vale la pena verificar si ya existe un paquete oficial o ampliamente adoptado que lo resuelva.

Un criterio importante al elegir un paquete de terceros es su mantenimiento activo, compatibilidad con la version actual de Nest, y cuantos otros proyectos serios lo usan, para evitar depender de algo que se abandona.

## Modelo mental

Pensa el ecosistema de NestJS como una caja de herramientas ya organizada: antes de fabricar tu propia herramienta para un problema comun, vale la pena revisar si ya hay una herramienta estandar, bien probada por miles de proyectos, que resuelve exactamente eso.

## Ejemplo base

```
Paquetes oficiales clave:
@nestjs/config        -> configuracion y variables de entorno
@nestjs/swagger        -> documentacion OpenAPI
@nestjs/passport + @nestjs/jwt -> autenticacion
@nestjs/throttler       -> rate limiting
@nestjs/schedule        -> cron jobs
@nestjs/bullmq          -> colas
@nestjs/event-emitter   -> eventos de dominio
@nestjs/terminus        -> health checks
@nestjs/cache-manager    -> caching

Terceros ampliamente usados:
class-validator / class-transformer -> validacion y serializacion
helmet                  -> seguridad HTTP
@casl/ability            -> autorizacion granular
nestjs-pino              -> logging estructurado de alto rendimiento
```

## Como se usa en un proyecto real

1. Antes de escribir una solucion custom, se busca si existe un paquete oficial (`@nestjs/*`) que resuelva el problema, priorizandolo por su integracion nativa.
2. Para necesidades no cubiertas oficialmente, se evalua un paquete de terceros por mantenimiento activo, numero de usuarios y compatibilidad con la version de Nest en uso.
3. Se documenta en el README del proyecto por que se eligio cada paquete no trivial, para que decisiones futuras de reemplazo tengan contexto.
4. Se evita sumar dependencias para resolver algo trivial que se implementaria en pocas lineas propias sin mantenimiento externo.
5. Se revisan periodicamente las dependencias del proyecto por paquetes abandonados que deban reemplazarse (ver capitulo de gestion de dependencias).

## Criterio junior

Un junior debe conocer los paquetes oficiales mas comunes (`@nestjs/config`, `@nestjs/swagger`, `@nestjs/passport`) y saber cuando buscar una solucion existente antes de escribir una propia.

## Criterio mid

Un mid debe evaluar criticamente un paquete de terceros antes de adoptarlo (mantenimiento, comunidad, compatibilidad) y justificar su eleccion frente a alternativas.

## Criterio senior

Un senior debe curar el set de dependencias del proyecto con criterio a largo plazo, anticipando el costo de mantener una dependencia en el tiempo, y liderar decisiones de reemplazo cuando un paquete clave queda abandonado.

## Errores comunes

- Escribir una solucion custom para un problema ya resuelto por un paquete oficial bien mantenido.
- Adoptar un paquete de terceros sin verificar su nivel de mantenimiento activo, quedando expuesto si se abandona.
- Sumar una dependencia pesada para resolver algo trivial que se implementaria en pocas lineas.
- No documentar por que se eligio un paquete no trivial, dificultando decisiones futuras de reemplazo.

## Practica

- [ ] Lista los paquetes oficiales `@nestjs/*` que usa (o deberia usar) un proyecto real que conozcas.
- [ ] Evalua un paquete de terceros que uses, revisando su mantenimiento activo y adopcion.
- [ ] Identifica una solucion custom en tu proyecto que podria reemplazarse por un paquete oficial existente.
- [ ] Documenta en un README por que se eligio un paquete no trivial de tu proyecto.
- [ ] Anota un criterio propio para decidir cuando sumar una nueva dependencia.

## Preguntas de repaso

- Por que conviene revisar el ecosistema oficial antes de escribir una solucion custom?
- Que criterios usarias para evaluar si un paquete de terceros es seguro de adoptar?
- Que riesgo tiene sumar una dependencia pesada para algo trivial?
- Como documentarias la eleccion de un paquete no trivial para que futuras decisiones tengan contexto?
