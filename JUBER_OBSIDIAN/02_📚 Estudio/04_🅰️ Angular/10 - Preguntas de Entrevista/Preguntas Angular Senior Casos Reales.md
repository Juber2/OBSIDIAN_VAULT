---
tags:
  - angular
  - entrevista
  - casos
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Preguntas de Entrevista Angular
  - ANGULAR
---

# Preguntas Angular Senior Casos Reales

## Preguntas clave

1. Cómo migrar Angular legacy con NgModules.
2. Cómo partir componente gigante.
3. Cómo diseñar permisos por rol.
4. Cómo mejorar testing sin paralizar al equipo.
5. Cómo decidir microfrontends.

## Respuesta senior esperada

Senior no responde “usa patrón X”. Propone pasos seguros, medición, cobertura mínima, ADRs y reducción de riesgo.

## Mini caso

Una app Angular antigua tiene servicios gigantes, cero tests y componentes de 1500 líneas. Propón plan de 30 días.

## Relacionado

- [[Preguntas de Entrevista Angular]]
- [[ANGULAR]]

## Cómo responder bien

Para **Preguntas Angular Senior Casos Reales**, evita respuestas de memoria. Una buena respuesta tiene cuatro partes: definición breve, ejemplo real, error común y criterio de decisión. Si te preguntan por una API de Angular, no basta decir qué hace; explica cuándo conviene usarla y cuándo puede complicar el proyecto.

## Formato recomendado

- Definición: una frase clara.
- Ejemplo: un caso concreto de app empresarial.
- Riesgo: qué sale mal si se usa mal.
- Tradeoff: alternativa posible y por qué elegirías una u otra.

## Ejemplo de enfoque

Si preguntan por Signals vs RxJS, una respuesta floja sería decir que Signals reemplaza RxJS. Una respuesta mejor: Signals modela estado actual y derivado; RxJS modela eventos en el tiempo, cancelación y composición async. En un buscador usaría RxJS para debounce y switchMap, y Signals para estado visual como selectedUser o isPanelOpen.

## Práctica oral

Responde cada pregunta en menos de dos minutos. Luego repítela con un ejemplo de código o arquitectura. Si no puedes dar un ejemplo, probablemente todavía no lo entendiste de forma usable.
