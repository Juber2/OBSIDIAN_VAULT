---
tags:
  - angular
  - entrevista
  - senior
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Preguntas de Entrevista Angular
  - ANGULAR
---

# Preguntas Arquitectura Performance Seguridad

## Preguntas clave

1. Feature architecture y boundaries.
2. Core vs Shared vs Features.
3. Cuándo usar SSR.
4. Cómo aplicar @defer.
5. Cómo manejar JWT y refresh tokens en frontend.
6. Cómo evitar XSS.

## Respuesta senior esperada

Hablar de tradeoffs, medición, backend como autoridad de seguridad, y costos de complejidad.

## Mini caso

Dashboard privado pesado. Decide entre CSR, SSR, lazy loading y @defer.

## Relacionado

- [[Preguntas de Entrevista Angular]]
- [[ANGULAR]]

## Cómo responder bien

Para **Preguntas Arquitectura Performance Seguridad**, evita respuestas de memoria. Una buena respuesta tiene cuatro partes: definición breve, ejemplo real, error común y criterio de decisión. Si te preguntan por una API de Angular, no basta decir qué hace; explica cuándo conviene usarla y cuándo puede complicar el proyecto.

## Formato recomendado

- Definición: una frase clara.
- Ejemplo: un caso concreto de app empresarial.
- Riesgo: qué sale mal si se usa mal.
- Tradeoff: alternativa posible y por qué elegirías una u otra.

## Ejemplo de enfoque

Si preguntan por Signals vs RxJS, una respuesta floja sería decir que Signals reemplaza RxJS. Una respuesta mejor: Signals modela estado actual y derivado; RxJS modela eventos en el tiempo, cancelación y composición async. En un buscador usaría RxJS para debounce y switchMap, y Signals para estado visual como selectedUser o isPanelOpen.

## Práctica oral

Responde cada pregunta en menos de dos minutos. Luego repítela con un ejemplo de código o arquitectura. Si no puedes dar un ejemplo, probablemente todavía no lo entendiste de forma usable.
