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

# Angular Material CDK Design System

## Material

Componentes listos con accesibilidad base y theming.

## CDK

Primitivas sin estilo: overlay, portal, drag-drop, virtual scroll, a11y.

## Design System

No es solo botones. Incluye tokens, estados, componentes, documentación y reglas de uso.

## Ejemplo


```html
<mat-form-field>
  <mat-label>Email</mat-label>
  <input matInput [formControl]="email" />
</mat-form-field>
```


## Criterio

No acoples dominio a Material. Envuelve componentes si necesitas consistencia o cambios futuros.

## Errores comunes

- Usar Material como arquitectura
- Componentes visuales con reglas de negocio
- No definir estados disabled/error/loading

## Práctica

- [ ] Crear componente AppButton documentado

## Relacionado

- [[Testing Ecosistema Senior Angular]]
- [[Angular - Roadmap]]

## Profundización

En **Angular Material CDK Design System**, piensa en riesgo. No todas las líneas merecen test, pero todo comportamiento crítico debería estar protegido: login, permisos, formularios, errores HTTP, mappers y navegación principal.

## Estrategia recomendada

- Unit tests para lógica pura, pipes, mappers y validators.
- Component tests para inputs, outputs y DOM visible.
- Integration tests para servicios HTTP, guards e interceptors.
- E2E para flujos críticos: login, CRUD principal y navegación.

## Criterio senior

Un test bueno falla por una regresión real, no por refactor interno. Evita probar detalles privados. Prefiere verificar comportamiento observable: texto visible, evento emitido, request hecha, ruta bloqueada o mensaje de error mostrado.
