---
tags:
  - Angular
  - Senior
  - Senior
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Arquitectura Performance Seguridad Angular
  - ANGULAR
---

# Facade Repository Adapter

## Facade

Expone estado y acciones simples para la UI.

## Repository

Coordina acceso a datos, cache y API.

## Adapter

Traduce DTO externo a modelo interno.

## Flujo


```text
UsersPage -> UsersFacade -> UsersRepository -> UsersApiService -> Backend
                                  -> UserAdapter -> User
```


## Cuándo usar

Cuando hay mapping, cache, estado o varias fuentes de datos. Para CRUD mínimo, un API service puede bastar.

## Errores comunes

- Facade que solo reexpone métodos
- Repository para todo sin valor
- Adapters mezclados en componentes

## Práctica

- [ ] Crear adapter UserDto -> User

## Relacionado

- [[Arquitectura Performance Seguridad Angular]]
- [[Angular - Roadmap]]

## Decisión técnica

En **Facade Repository Adapter**, no busques una receta única. Angular senior consiste en elegir estructura según tamaño del equipo, vida útil del producto, cantidad de features, necesidad de SEO, performance, permisos y testing.

## Preguntas antes de decidir

- ¿Este código cambiará por negocio o por infraestructura?
- ¿La feature tiene ownership claro?
- ¿Necesita estado compartido o solo local?
- ¿Hay rutas lazy y boundaries?
- ¿Qué parte debe poder testearse sin levantar toda la app?

## Criterio de madurez

Una arquitectura buena no se ve solo en carpetas bonitas. Se ve cuando agregar una feature no rompe otra, cuando una PR es revisable, cuando los tests cubren riesgos y cuando el equipo entiende dónde poner cada pieza.
