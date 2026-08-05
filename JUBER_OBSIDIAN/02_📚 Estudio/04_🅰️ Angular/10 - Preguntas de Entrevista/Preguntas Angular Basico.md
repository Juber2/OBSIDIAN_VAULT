---
tags:
  - angular
  - entrevista
  - basico
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Preguntas de Entrevista Angular
  - ANGULAR
---

# Preguntas Angular Basico

## Preguntas clave

1. Qué problema resuelve Angular.
2. Qué es un standalone component.
3. Diferencia entre input, output y model input.
4. Property binding vs event binding.
5. Para qué sirve `track` en `@for`.

## Respuesta senior esperada

Aunque sean básicas, responde conectando con mantenibilidad: dependencias explícitas, componentes pequeños, templates legibles y render eficiente.

## Mini caso

Tienes una lista que parpadea y pierde estado al actualizarse. Explica cómo revisarías el `track`.

## Relacionado

- [[Preguntas de Entrevista Angular]]
- [[ANGULAR]]

## Cómo responder bien

Para **Preguntas Angular Basico**, evita respuestas de memoria. Una buena respuesta tiene cuatro partes: definición breve, ejemplo real, error común y criterio de decisión. Si te preguntan por una API de Angular, no basta decir qué hace; explica cuándo conviene usarla y cuándo puede complicar el proyecto.

## Formato recomendado

- Definición: una frase clara.
- Ejemplo: un caso concreto de app empresarial.
- Riesgo: qué sale mal si se usa mal.
- Tradeoff: alternativa posible y por qué elegirías una u otra.

## Ejemplo de enfoque

Si preguntan por Signals vs RxJS, una respuesta floja sería decir que Signals reemplaza RxJS. Una respuesta mejor: Signals modela estado actual y derivado; RxJS modela eventos en el tiempo, cancelación y composición async. En un buscador usaría RxJS para debounce y switchMap, y Signals para estado visual como selectedUser o isPanelOpen.

## Práctica oral

Responde cada pregunta en menos de dos minutos. Luego repítela con un ejemplo de código o arquitectura. Si no puedes dar un ejemplo, probablemente todavía no lo entendiste de forma usable.
