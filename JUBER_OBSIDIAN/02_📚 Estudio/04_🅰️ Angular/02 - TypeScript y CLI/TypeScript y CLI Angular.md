---
tags:
  - Angular
  - Básico
Creado: 2026-05-29
Nivel: Básico
Estado: Pendiente
Relacionado:
  - ANGULAR
---

# TypeScript y CLI Angular

## Objetivo

Dominar tipos, convenciones y estructura de proyecto.

## Objetivos de aprendizaje

- [ ] Tipar modelos y DTOs
- [ ] Usar generics
- [ ] Crear proyecto con CLI
- [ ] Entender archivos de configuración

## Temas

| # | Tema | Estado | Nota |
|---|---|---|---|
| 1 | TypeScript esencial para Angular | ⬜ | [[TypeScript esencial para Angular]] |
| 2 | Convenciones de nombres Angular | ⬜ | [[Convenciones de nombres Angular]] |
| 3 | Angular CLI y estructura | ⬜ | [[Angular CLI y estructura]] |

## Práctica del módulo

- [ ] Crear UserDto/User y mapper
- [ ] Crear proyecto Angular con routing
- [ ] Generar componente/servicio/guard

## Errores comunes del módulo

- Usar any
- No separar DTO y modelo
- Editar configuración sin entender

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

## Aplicación profesional

En **TypeScript y CLI Angular**, TypeScript no es decoración. Es una herramienta para hacer explícitos contratos, reducir errores y mejorar refactors. Un buen tipo revela intención: request, response, model, form value, route data, state o permission.

## Reglas útiles

- No uses any para respuestas HTTP.
- Separa DTO externo de modelo interno cuando el backend no calza con la UI.
- Usa unions para estados finitos: loading, success, error.
- Usa nombres que indiquen rol del archivo: service, store, guard, interceptor, adapter.

## Estado tipado

Un estado tipado obliga al template a tratar cada caso de forma explícita. Esto reduce pantallas con ifs ambiguos y evita errores donde la UI intenta leer data cuando todavía está cargando o falló.
