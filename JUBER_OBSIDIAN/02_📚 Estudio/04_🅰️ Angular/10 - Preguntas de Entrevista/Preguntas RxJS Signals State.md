---
tags:
  - angular
  - entrevista
  - reactividad
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Preguntas de Entrevista Angular
  - ANGULAR
---

# Preguntas RxJS Signals State

## Preguntas clave

1. Observable vs Signal.
2. Cuándo usar switchMap, mergeMap, concatMap y exhaustMap.
3. computed vs effect.
4. BehaviorSubject vs Signal.
5. Cuándo usar NgRx.

## Respuesta senior esperada

Signals representan estado actual; RxJS representa eventos en el tiempo. La respuesta debe mencionar cancelación, side effects, estado derivado y complejidad del equipo.

## Mini caso

Buscador con debounce y cancelación. Justifica RxJS para búsqueda y Signals para estado de UI.

## Relacionado

- [[Preguntas de Entrevista Angular]]
- [[ANGULAR]]

## Cómo responder bien

Para **Preguntas RxJS Signals State**, evita respuestas de memoria. Una buena respuesta tiene cuatro partes: definición breve, ejemplo real, error común y criterio de decisión. Si te preguntan por una API de Angular, no basta decir qué hace; explica cuándo conviene usarla y cuándo puede complicar el proyecto.

## Formato recomendado

- Definición: una frase clara.
- Ejemplo: un caso concreto de app empresarial.
- Riesgo: qué sale mal si se usa mal.
- Tradeoff: alternativa posible y por qué elegirías una u otra.

## Ejemplo de enfoque

Si preguntan por Signals vs RxJS, una respuesta floja sería decir que Signals reemplaza RxJS. Una respuesta mejor: Signals modela estado actual y derivado; RxJS modela eventos en el tiempo, cancelación y composición async. En un buscador usaría RxJS para debounce y switchMap, y Signals para estado visual como selectedUser o isPanelOpen.

## Práctica oral

Responde cada pregunta en menos de dos minutos. Luego repítela con un ejemplo de código o arquitectura. Si no puedes dar un ejemplo, probablemente todavía no lo entendiste de forma usable.
