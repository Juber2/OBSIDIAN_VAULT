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

# Convenciones de nombres Angular

## Tabla rápida

| Elemento | Convención | Ejemplo |
|---|---|---|
| Clase | PascalCase | UserService |
| Interface/type | PascalCase | UserProfile |
| Variable/método | camelCase | isLoading |
| Archivo | kebab-case | user-card.component.ts |
| Selector | kebab-case | app-user-card |
| Constante global | UPPER_SNAKE_CASE | API_BASE_URL |

## Ejemplo completo

`user-list.page.ts` -> `UserListPage`
`user-card.component.ts` -> `UserCardComponent`
`user.service.ts` -> `UserService`
`user-role.enum.ts` -> `UserRole`

## Criterio

El nombre debe revelar rol: page, component, service, guard, interceptor, store, facade, adapter.

## Errores comunes

- Mezclar camelCase en archivos
- Usar nombres genéricos como data.service.ts
- Selectores sin prefijo

## Práctica

- [ ] Renombrar una feature siguiendo convenciones

## Relacionado

- [[TypeScript y CLI Angular]]
- [[Angular - Roadmap]]

## Aplicación profesional

En **Convenciones de nombres Angular**, TypeScript no es decoración. Es una herramienta para hacer explícitos contratos, reducir errores y mejorar refactors. Un buen tipo revela intención: request, response, model, form value, route data, state o permission.

## Reglas útiles

- No uses any para respuestas HTTP.
- Separa DTO externo de modelo interno cuando el backend no calza con la UI.
- Usa unions para estados finitos: loading, success, error.
- Usa nombres que indiquen rol del archivo: service, store, guard, interceptor, adapter.

## Estado tipado

Un estado tipado obliga al template a tratar cada caso de forma explícita. Esto reduce pantallas con ifs ambiguos y evita errores donde la UI intenta leer data cuando todavía está cargando o falló.
