---
tags:
  - Angular
  - Intermedio
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Relacionado:
  - ANGULAR
---

# Servicios DI HTTP Angular

## Objetivo

Separar lógica, dependencias y comunicación backend.

## Objetivos de aprendizaje

- [ ] Diseñar servicios con responsabilidad clara
- [ ] Usar DI e InjectionToken
- [ ] Consumir APIs con HttpClient
- [ ] Crear interceptors funcionales

## Temas

| # | Tema | Estado | Nota |
|---|---|---|---|
| 1 | Servicios por responsabilidad | ⬜ | [[Servicios por responsabilidad]] |
| 2 | Dependency Injection e InjectionToken | ⬜ | [[Dependency Injection e InjectionToken]] |
| 3 | HttpClient DTOs mapping e interceptors | ⬜ | [[HttpClient DTOs mapping e interceptors]] |

## Práctica del módulo

- [ ] Crear UsersApiService
- [ ] Crear API_URL InjectionToken
- [ ] Crear auth interceptor

## Errores comunes del módulo

- HttpClient en componentes
- Servicios gigantes
- Interceptors con demasiada lógica

## Recursos oficiales

- Angular Docs: https://angular.dev/
- Components: https://angular.dev/guide/components
- Templates: https://angular.dev/guide/templates
- Signals: https://angular.dev/guide/signals
- Router: https://angular.dev/guide/routing
- Forms: https://angular.dev/guide/forms
- HTTP: https://angular.dev/guide/http
- Defer: https://angular.dev/guide/templates/defer
- Hydration: https://angular.dev/guide/hydration
- Testing: https://angular.dev/guide/testing

## Diseño aplicado

En **Servicios DI HTTP Angular**, el punto importante es separar responsabilidades. Angular permite inyectar casi cualquier cosa, pero eso no significa que un servicio deba concentrar API, estado, reglas de negocio, navegación y mensajes visuales.

## Separación recomendada

- API service: habla con backend y devuelve DTOs o modelos mapeados.
- Store/facade de feature: expone estado y acciones para componentes.
- Adapter/mapper: traduce DTO externo a modelo interno.
- Utility: función pura sin dependencia de Angular.

## Ejemplo de criterio

Si una función necesita HttpClient, va hacia data-access. Si necesita Router, probablemente pertenece a una capa de flujo o componente contenedor. Si solo transforma UserDto en User, debería ser una función pura testeable.

## Revisión rápida

Abre el servicio y pregunta: ¿por qué cambia este archivo? Si cambia por demasiadas razones, está acumulando responsabilidades.
