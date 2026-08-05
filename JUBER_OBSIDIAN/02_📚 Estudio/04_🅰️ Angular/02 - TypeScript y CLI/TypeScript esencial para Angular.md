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

# TypeScript esencial para Angular

## Interfaces y types

Usa interfaces/types para describir datos, props, DTOs, respuestas y estados.

## DTO vs modelo

El DTO es externo y puede venir en snake_case. El modelo de UI debe ser cómodo para Angular.

## Generics

Útiles para respuestas API y helpers reutilizables.

## Ejemplo


```ts
interface ApiResponse<T> {
  data: T;
  errors: string[];
}

interface UserDto { id: string; first_name: string; }
interface User { id: string; firstName: string; }

const toUser = (dto: UserDto): User => ({
  id: dto.id,
  firstName: dto.first_name
});
```


## Errores comunes

- Usar any en responses HTTP
- Tipar todo como string
- No validar datos externos

## Práctica

- [ ] Crear DTO, modelo y mapper para Course

## Relacionado

- [[TypeScript y CLI Angular]]
- [[Angular - Roadmap]]

## Aplicación profesional

En **TypeScript esencial para Angular**, TypeScript no es decoración. Es una herramienta para hacer explícitos contratos, reducir errores y mejorar refactors. Un buen tipo revela intención: request, response, model, form value, route data, state o permission.

## Reglas útiles

- No uses any para respuestas HTTP.
- Separa DTO externo de modelo interno cuando el backend no calza con la UI.
- Usa unions para estados finitos: loading, success, error.
- Usa nombres que indiquen rol del archivo: service, store, guard, interceptor, adapter.

## Estado tipado

Un estado tipado obliga al template a tratar cada caso de forma explícita. Esto reduce pantallas con ifs ambiguos y evita errores donde la UI intenta leer data cuando todavía está cargando o falló.
