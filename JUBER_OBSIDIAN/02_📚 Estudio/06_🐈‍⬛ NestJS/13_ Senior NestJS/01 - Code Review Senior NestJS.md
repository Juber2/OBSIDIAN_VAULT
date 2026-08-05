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

# 01 - Code Review Senior NestJS

## Que vas a aprender

En este capitulo vas a consolidar una checklist de **code review** especifica para NestJS, que va mas alla de sintaxis: separacion de capas, seguridad, performance y testing, aplicando de forma critica todo lo visto en los capitulos anteriores.

## Explicacion

Un code review senior no busca solo "que el codigo funcione" ni "que siga el estilo del linter" (eso ya lo automatiza ESLint/Prettier). Busca: que la separacion de capas se respete (controller delgado, logica en el servicio), que no se introduzcan riesgos de seguridad (validacion faltante, exposicion de datos sensibles, autorizacion no verificada), que no se degrade performance (N+1 nuevo, falta de indice, cache no invalidado), y que la cobertura de tests sea proporcional al riesgo del cambio, no solo un numero.

La forma de dar feedback tambien importa: un review senior distingue entre bloqueantes reales (bugs, riesgos de seguridad) y preferencias de estilo (que deberian resolverse con herramientas, no con comentarios humanos repetidos).

## Modelo mental

Pensa el code review como una ultima linea de defensa antes de que algo entre al codigo compartido: no reemplaza tests ni linters automaticos, se enfoca en lo que las maquinas no pueden evaluar todavia (si la solucion es la correcta para el problema real, si hay riesgos que el autor no vio).

## Ejemplo base

```
Checklist minima de review para un PR de NestJS:

[ ] El controller es delgado; la logica vive en el servicio.
[ ] Los DTOs validan correctamente los datos de entrada (class-validator).
[ ] Ningun campo sensible se filtra en la respuesta (serializacion revisada).
[ ] Autorizacion verificada, no solo autenticacion.
[ ] Sin queries dentro de loops (N+1).
[ ] Tests cubren el camino feliz y al menos un caso de error.
[ ] Sin secretos ni credenciales hardcodeadas.
[ ] Migrations incluidas si hay cambio de esquema, con `down` funcional.
```

## Como se usa en un proyecto real

1. Se usa una checklist compartida (adaptada de temas anteriores) como base minima de cualquier review, sin depender de la memoria de cada reviewer.
2. Se automatiza todo lo automatizable (lint, formato, tests, cobertura minima) para que el review humano se enfoque en logica y diseño.
3. Se distingue explicitamente entre comentarios bloqueantes y sugerencias opcionales, evitando ambiguedad sobre que debe resolverse antes de mergear.
4. Se revisa con especial atencion cualquier cambio que toque autenticacion, autorizacion, manejo de datos sensibles o cambios de esquema de base de datos.
5. Se da feedback constructivo explicando el "por que", no solo el "que", para que el review tambien sea una oportunidad de aprendizaje.

## Criterio junior

Un junior debe poder seguir una checklist de review dada y detectar problemas obvios (falta de validacion, controller con logica de negocio).

## Criterio mid

Un mid debe hacer reviews proactivos detectando N+1, problemas de autorizacion, y cobertura de tests insuficiente sin necesitar que se lo señalen explicitamente.

## Criterio senior

Un senior debe liderar la cultura de code review del equipo, definir la checklist compartida, mentorear a traves del feedback, y saber cuando aprobar con sugerencias menores versus cuando bloquear por un riesgo real.

## Errores comunes

- Enfocar el review en estilo de codigo que ya deberia resolver el linter, ignorando riesgos reales de seguridad o performance.
- Aprobar un PR sin revisar tests, o revisando solo que "existan" sin evaluar si cubren los casos relevantes.
- Dar feedback ambiguo sin distinguir que es bloqueante y que es sugerencia opcional.
- No revisar con especial cuidado cambios en autenticacion, autorizacion o esquema de base de datos, tratandolos igual que cualquier otro cambio.

## Practica

- [ ] Define una checklist de review compartida para tu equipo, basada en los capitulos de este libro.
- [ ] Revisa un PR real (o simulado) aplicando esa checklist.
- [ ] Identifica en un PR existente un problema de N+1, de autorizacion, o de exposicion de datos sensibles.
- [ ] Practica distinguir explicitamente comentarios bloqueantes de sugerencias en un review.
- [ ] Anota que automatizaciones (lint, tests, cobertura) faltan en tu pipeline para liberar tiempo de review humano.

## Preguntas de repaso

- Que deberia estar automatizado (linter, CI) en vez de depender de un review humano?
- Que revisarias con especial cuidado en un PR que toca autenticacion o autorizacion?
- Como distinguirias un comentario bloqueante de una sugerencia opcional en un review?
- Que señal te indicaria que la cobertura de tests de un PR es insuficiente, mas alla del numero de cobertura?
