---
tags:
  - Java
  - Spring Boot
  - Proyecto
Creado: 2026-05-21
Relacionado:
  - Proyectos Java
---

# Proyecto API CRUD

## Objetivo

Crear una API REST de productos con Spring Boot.

## Requisitos

- CRUD de productos.
- Validaciones.
- Manejo global de errores.
- DTOs request/response.
- Tests de controller y service.

## Endpoints

| Metodo | Ruta | Uso |
|---|---|---|
| GET | `/products` | listar |
| GET | `/products/{id}` | obtener |
| POST | `/products` | crear |
| PUT | `/products/{id}` | actualizar |
| DELETE | `/products/{id}` | eliminar |

## Checklist

- [ ] No expongo entidades JPA.
- [ ] Hay respuestas HTTP correctas.
- [ ] Hay errores consistentes.
- [ ] Hay tests.
