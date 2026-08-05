---
tags:
  - Angular
  - Senior
  - Testing
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Testing Ecosistema Senior Angular
  - ANGULAR
---

# Code Review Senior Angular

## Qué revisar

Responsabilidad, boundaries, estado, tipado, errores, performance, accesibilidad, seguridad y pruebas.

## Checklist

- ¿El componente hace demasiado?
- ¿Hay loading/error/empty?
- ¿Los imports respetan features?
- ¿Se usa track en listas?
- ¿Hay streams sin limpiar?
- ¿Backend valida permisos?
- ¿Hay tests donde hay riesgo?

## Cómo comentar

Un buen comentario explica riesgo y alternativa. No se queda en gusto personal.

## Ejemplo de comentario

“Este componente está llamando API y formateando DTOs. Sugiero mover HTTP a data-access y dejar aquí solo estado de UI para que sea testeable.”

## Errores comunes

- Reviews solo de estilo
- No explicar tradeoff
- Pedir abstracciones sin necesidad

## Práctica

- [ ] Revisar una feature propia con checklist

## Relacionado

- [[Testing Ecosistema Senior Angular]]
- [[Angular - Roadmap]]

## Profundización

En **Code Review Senior Angular**, piensa en riesgo. No todas las líneas merecen test, pero todo comportamiento crítico debería estar protegido: login, permisos, formularios, errores HTTP, mappers y navegación principal.

## Estrategia recomendada

- Unit tests para lógica pura, pipes, mappers y validators.
- Component tests para inputs, outputs y DOM visible.
- Integration tests para servicios HTTP, guards e interceptors.
- E2E para flujos críticos: login, CRUD principal y navegación.

## Criterio senior

Un test bueno falla por una regresión real, no por refactor interno. Evita probar detalles privados. Prefiere verificar comportamiento observable: texto visible, evento emitido, request hecha, ruta bloqueada o mensaje de error mostrado.
