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

# Testing Strategy Angular

## Pirámide práctica

| Nivel | Uso |
|---|---|
| Unit | lógica pura, mappers, pipes |
| Component | inputs, outputs, DOM visible |
| Integration | HTTP/router/guards/interceptors |
| E2E | flujos críticos |

## Component test


```ts
const fixture = TestBed.createComponent(UserCardComponent);
fixture.componentRef.setInput('user', user);
fixture.detectChanges();
expect(fixture.nativeElement.textContent).toContain(user.name);
```


## HTTP test


```ts
const req = httpMock.expectOne('/api/users');
expect(req.request.method).toBe('GET');
req.flush([{ id: '1' }]);
```


## Qué testear

Reglas, contratos, errores, permisos, formularios, mappers, flujos críticos.

## Errores comunes

- Tests que repiten implementación
- No probar errores
- Demasiados E2E lentos

## Práctica

- [ ] Crear test de componente y servicio HTTP

## Relacionado

- [[Testing Ecosistema Senior Angular]]
- [[Angular - Roadmap]]

## Profundización

En **Testing Strategy Angular**, piensa en riesgo. No todas las líneas merecen test, pero todo comportamiento crítico debería estar protegido: login, permisos, formularios, errores HTTP, mappers y navegación principal.

## Estrategia recomendada

- Unit tests para lógica pura, pipes, mappers y validators.
- Component tests para inputs, outputs y DOM visible.
- Integration tests para servicios HTTP, guards e interceptors.
- E2E para flujos críticos: login, CRUD principal y navegación.

## Criterio senior

Un test bueno falla por una regresión real, no por refactor interno. Evita probar detalles privados. Prefiere verificar comportamiento observable: texto visible, evento emitido, request hecha, ruta bloqueada o mensaje de error mostrado.

## Plan de cobertura por riesgo

Una estrategia sana no empieza preguntando cuanto coverage quieres, sino que comportamiento dolería romper. En Angular, normalmente los riesgos fuertes están en autenticación, permisos, formularios, contratos HTTP, transformación de DTOs, navegación y componentes compartidos usados por muchas pantallas.

Para una feature nueva, una cobertura razonable puede ser:

- Mapper o adapter con unit tests.
- Servicio HTTP con request, params, headers y manejo de error.
- Componente principal con loading, empty, error y success.
- Guard o interceptor si la feature depende de sesión.
- Un E2E solo para el flujo crítico, no para cada variante visual.

## Anti patrón frecuente

Un error común es escribir muchos tests de snapshot o tests que verifican detalles internos del componente, pero no cubren el comportamiento que el usuario o el negocio necesita. Si cambias una implementación interna y se rompen diez tests aunque la pantalla sigue funcionando igual, la suite está demasiado acoplada.

La pregunta correcta para cada test es: si esto falla en producción, ¿este test lo habría detectado?
