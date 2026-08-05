---
tags:
  - angular
  - indice
Creado: 2026-05-30
Estado: Pendiente
Relacionado:
  - ANGULAR
---

# Preguntas de Entrevista Angular

## Objetivo

Preparar respuestas con criterio, ejemplos y tradeoffs sobre Angular moderno.

## Contenido

| # | Nota | Estado |
|---|---|---|
| 1 | [[Preguntas Angular Basico]] | ⬜ |
| 2 | [[Preguntas Angular Intermedio]] | ⬜ |
| 3 | [[Preguntas RxJS Signals State]] | ⬜ |
| 4 | [[Preguntas Arquitectura Performance Seguridad]] | ⬜ |
| 5 | [[Preguntas Angular Senior Casos Reales]] | ⬜ |

## Practica recomendada

- [ ] Responder cada pregunta con ejemplo
- [ ] Agregar error común
- [ ] Explicar cuándo no usar la técnica

## Cómo responder bien

Para **Preguntas de Entrevista Angular**, evita respuestas de memoria. Una buena respuesta tiene cuatro partes: definición breve, ejemplo real, error común y criterio de decisión. Si te preguntan por una API de Angular, no basta decir qué hace; explica cuándo conviene usarla y cuándo puede complicar el proyecto.

## Formato recomendado

- Definición: una frase clara.
- Ejemplo: un caso concreto de app empresarial.
- Riesgo: qué sale mal si se usa mal.
- Tradeoff: alternativa posible y por qué elegirías una u otra.

## Ejemplo de enfoque

Si preguntan por Signals vs RxJS, una respuesta floja sería decir que Signals reemplaza RxJS. Una respuesta mejor: Signals modela estado actual y derivado; RxJS modela eventos en el tiempo, cancelación y composición async. En un buscador usaría RxJS para debounce y switchMap, y Signals para estado visual como selectedUser o isPanelOpen.

## Práctica oral

Responde cada pregunta en menos de dos minutos. Luego repítela con un ejemplo de código o arquitectura. Si no puedes dar un ejemplo, probablemente todavía no lo entendiste de forma usable.
