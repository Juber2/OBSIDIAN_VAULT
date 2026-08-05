---
tags:
  - angular
  - entrevista
  - intermedio
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Preguntas de Entrevista Angular
  - ANGULAR
---

# Preguntas Angular Intermedio

## Preguntas clave

1. Cómo funciona Dependency Injection.
2. providedIn root vs provider en componente.
3. Cuándo usar guard y resolver.
4. Typed Reactive Forms.
5. Interceptors funcionales y separación de responsabilidades.

## Señales de criterio

Diferenciar seguridad real backend de protección visual frontend, y explicar scope de providers sin memorizar definiciones.

## Mini caso

Un servicio mantiene estado duplicado por pantalla. Explica cómo el provider scope puede causar eso.

## Relacionado

- [[Preguntas de Entrevista Angular]]
- [[ANGULAR]]

## Cómo responder bien

Para **Preguntas Angular Intermedio**, evita respuestas de memoria. Una buena respuesta tiene cuatro partes: definición breve, ejemplo real, error común y criterio de decisión. Si te preguntan por una API de Angular, no basta decir qué hace; explica cuándo conviene usarla y cuándo puede complicar el proyecto.

## Formato recomendado

- Definición: una frase clara.
- Ejemplo: un caso concreto de app empresarial.
- Riesgo: qué sale mal si se usa mal.
- Tradeoff: alternativa posible y por qué elegirías una u otra.

## Ejemplo de enfoque

Si preguntan por Signals vs RxJS, una respuesta floja sería decir que Signals reemplaza RxJS. Una respuesta mejor: Signals modela estado actual y derivado; RxJS modela eventos en el tiempo, cancelación y composición async. En un buscador usaría RxJS para debounce y switchMap, y Signals para estado visual como selectedUser o isPanelOpen.

## Práctica oral

Responde cada pregunta en menos de dos minutos. Luego repítela con un ejemplo de código o arquitectura. Si no puedes dar un ejemplo, probablemente todavía no lo entendiste de forma usable.
