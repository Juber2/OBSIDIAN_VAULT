---
tags:
  - Angular
  - Básico
  - TypeScript
Creado: 2026-05-29
Nivel: Básico
Estado: Pendiente
Prioridad: Media
Relacionado:
  - TypeScript y CLI Angular
  - ANGULAR
---

# Angular CLI y estructura

## Crear proyecto


```bash
ng new staff-training-web --routing --style=scss
```


## Archivos clave

| Archivo | Rol |
|---|---|
| angular.json | build, assets, styles, budgets |
| package.json | scripts y dependencias |
| tsconfig.json | compilación TypeScript |
| main.ts | arranque |
| app.config.ts | providers globales |
| app.routes.ts | rutas raíz |

## Generadores útiles


```bash
ng g c features/users/ui/user-card
ng g s features/users/data-access/users-api
ng g guard core/auth/auth
ng g interceptor core/http/auth
```


## Estructura inicial recomendada


```text
src/app/
  core/
  shared/
  features/
    users/
    auth/
  app.config.ts
  app.routes.ts
```


## Errores comunes

- Generar todo en app/ sin features
- No revisar angular.json
- Instalar paquetes sin criterio

## Práctica

- [ ] Crear app y primera feature users

## Relacionado

- [[TypeScript y CLI Angular]]
- [[Angular - Roadmap]]

## Aplicación profesional

En **Angular CLI y estructura**, TypeScript no es decoración. Es una herramienta para hacer explícitos contratos, reducir errores y mejorar refactors. Un buen tipo revela intención: request, response, model, form value, route data, state o permission.

## Reglas útiles

- No uses any para respuestas HTTP.
- Separa DTO externo de modelo interno cuando el backend no calza con la UI.
- Usa unions para estados finitos: loading, success, error.
- Usa nombres que indiquen rol del archivo: service, store, guard, interceptor, adapter.

## Estado tipado

Un estado tipado obliga al template a tratar cada caso de forma explícita. Esto reduce pantallas con ifs ambiguos y evita errores donde la UI intenta leer data cuando todavía está cargando o falló.
