---
tags:
  - Angular
  - Básico
  - Base
Creado: 2026-05-29
Nivel: Básico
Estado: Pendiente
Prioridad: Media
Relacionado:
  - Requisitos previos Angular
  - ANGULAR
---

# HTTP APIs REST y JSON

## REST desde Angular

Angular consume recursos expuestos por backend. Los verbos HTTP expresan intención: GET lee, POST crea, PUT reemplaza, PATCH actualiza parcial y DELETE elimina.

## Status codes que importan

200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 500 Internal Server Error.

## JSON

JSON es formato de intercambio, no tu modelo interno. Conviene mapear DTOs a modelos de UI.

## Ejemplo de contrato


```http
GET /api/v1/users?page=1
Authorization: Bearer <token>

200 OK
{ "data": [], "total": 0 }
```


## Errores comunes

- Tratar todos los errores como 500
- No diferenciar 401 y 403
- Enviar filtros complejos en body para GET

## Práctica

- [ ] Diseñar contrato de endpoint para listar usuarios

## Relacionado

- [[Requisitos previos Angular]]
- [[Angular - Roadmap]]

## Por qué importa antes de Angular

En **HTTP APIs REST y JSON**, el objetivo es evitar que Angular cargue con problemas que realmente vienen de bases web. Si no entiendes HTML, HTTP, JavaScript async o CSS, terminarás culpando al framework por errores de contrato, layout o asincronía.

## Señales de dominio

- Puedes explicar qué pasa entre click, request, response y render.
- Distingues error de red, 401, 403, 404 y 500.
- Sabes cuándo un problema es CSS, template, estado o API.
- Puedes leer JSON y mapearlo a un modelo de UI.

## Mini práctica

Antes de resolverlo en Angular, intenta hacer una versión mínima con HTML, TypeScript o fetch. Si entiendes la versión simple, Angular se vuelve una herramienta; si no, Angular se vuelve una caja negra.
