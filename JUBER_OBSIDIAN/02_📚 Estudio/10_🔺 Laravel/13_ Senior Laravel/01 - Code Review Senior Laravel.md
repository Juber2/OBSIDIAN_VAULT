---
tags:
  - Laravel
  - PHP
  - Senior-Laravel-Engineer
Creado: 2026-05-28
Relacionado:
  - Senior Laravel Engineer
  - LARAVEL
---

# 01 - Code Review Senior Laravel

## Que vas a aprender

En este capitulo vas a estudiar **Code Review Senior Laravel** dentro de Laravel, desde la idea central hasta los criterios que importan cuando el proyecto crece. La meta no es memorizar comandos: es poder tomar decisiones tecnicas y reconocer cuando una implementacion esta quedando fragil.

## Explicacion

Code Review Senior Laravel es una pieza del bloque **Senior Laravel Engineer**. No se estudia como definicion aislada, sino como herramienta para construir aplicaciones Laravel mantenibles. La pregunta importante no es solo como se usa, sino donde conviene ponerlo, que problema evita y que costo introduce.

La seniority aparece en decisiones: que abstraer, que dejar simple, como revisar codigo, como reducir riesgo y como guiar al equipo con estandares sanos.

Laravel te da velocidad porque muchas decisiones ya vienen integradas: estructura de carpetas, nombres, comandos, configuracion, ORM, colas, eventos, testing y despliegue. El peligro aparece cuando esa velocidad se convierte en mezclar responsabilidades. Por eso cada tema debe responder tres preguntas: que capa toca, que responsabilidad tiene y como se prueba.

## Modelo mental

Piensalo asi: cada feature Laravel atraviesa varias capas. Primero llega como request, luego pasa por middleware y validacion, despues entra a una regla de aplicacion o consulta de datos, y finalmente vuelve como response, vista, job o evento. Si mezclas todas esas responsabilidades en un solo archivo, el sistema funciona al inicio pero se vuelve dificil de cambiar.

En Laravel, la convencion suele darte velocidad, pero la seniority consiste en saber cuando seguirla literalmente y cuando crear una separacion extra. Este tema debe estudiarse con codigo real, tests y revision de casos borde.

## Ejemplo base

```php
// Pregunta senior: que pasa si este job se ejecuta dos veces?
// Si la respuesta rompe datos, falta idempotencia.
```

Este ejemplo no es una receta final. Es una forma de ubicar el concepto en codigo. En un proyecto real debes ajustar nombres, validaciones, permisos, transacciones y tests segun el dominio.

## Como se usa en un proyecto real

1. Define el caso de uso antes de escribir la clase o comando.
2. Decide que parte pertenece al framework y que parte pertenece al dominio.
3. Evita que el controlador sea el lugar donde vive todo.
4. Agrega validacion y autorizacion cerca de la entrada HTTP.
5. Mide o prueba los puntos que puedan fallar: base de datos, permisos, jobs, errores externos o concurrencia.

## Criterio junior

Un junior debe poder usar el feature siguiendo la documentacion, entender el comando o clase principal, y explicar que problema resuelve sin copiar una receta completa. Tambien debe saber donde esta el archivo generado y como conectarlo con una ruta, modelo, vista, job o test.

## Criterio mid

Un mid debe poder integrarlo con validacion, errores, base de datos, autorizacion y tests. Tambien debe reconocer tradeoffs: cuando usarlo, cuando evitarlo y que impacto tiene en performance o mantenimiento.

## Criterio senior

Un senior debe poder convertir este tema en una decision de arquitectura: definir convenciones para el equipo, prevenir abuso, revisar PRs, medir riesgos y documentar el criterio para que otros puedan mantenerlo.

## Errores comunes

- Copiar ejemplos sin entender en que capa viven.
- Meter logica de negocio en controllers, vistas, jobs o modelos sin criterio.
- No cubrir casos de error ni permisos.
- Ignorar performance hasta que aparezca en produccion.
- Resolver con paquetes externos algo que Laravel ya ofrece de forma nativa.

## Practica

- [ ] Implementa un ejemplo minimo relacionado con este tema.
- [ ] Agrega validacion o manejo de error.
- [ ] Escribe un test feature o unitario segun corresponda.
- [ ] Revisa si hay queries innecesarias, acoplamiento o duplicacion.
- [ ] Anota una decision o regla que aplicarias en un proyecto real.

## Preguntas de repaso

- Que problema resuelve este tema en Laravel?
- Que responsabilidad no deberia mezclarse aqui?
- Como lo probarias?
- Que podria salir mal en produccion?
- Que convencion de equipo definirias para usarlo bien?
