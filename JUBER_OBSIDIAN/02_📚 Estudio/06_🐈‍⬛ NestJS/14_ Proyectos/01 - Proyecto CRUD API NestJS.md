---
tags:
  - NestJS
  - TypeScript
  - Nest-Proyectos
Creado: 2026-07-27
Relacionado:
  - Proyectos
  - NESTJS
---

# 01 - Proyecto CRUD API NestJS

## Que vas a construir

Una API REST CRUD completa para un dominio simple (por ejemplo, un catalogo de productos o una lista de tareas), aplicando de punta a punta los capitulos de Fundamentos, NestJS Core, Base de Datos y APIs REST.

## Alcance funcional

- CRUD completo de un recurso principal (crear, listar con paginacion/filtros, obtener uno, actualizar, eliminar).
- Al menos una relacion (`OneToMany` o `ManyToOne`) con un segundo recurso.
- Validacion completa de entrada con DTOs y `class-validator`.
- Documentacion de la API con Swagger.
- Manejo centralizado de errores con un exception filter global.

## Checklist tecnica

- [ ] Estructura del proyecto organizada por dominio (ver capitulo de Nest CLI y Estructura de Proyecto).
- [ ] Entidades con relaciones correctamente modeladas, sin N+1 en el listado principal.
- [ ] Migrations versionadas para el esquema completo, sin `synchronize: true`.
- [ ] DTOs separados para create, update y response.
- [ ] Paginacion, filtros y ordenamiento con whitelist segura en el endpoint de listado.
- [ ] Exception filter global que nunca expone detalles internos en error 500.
- [ ] Documentacion Swagger completa, incluyendo respuestas de error.
- [ ] Tests unitarios del servicio principal (camino feliz y al menos un caso de error).

## Criterio de exito

El proyecto se considera completo cuando otra persona puede clonar el repositorio, levantarlo siguiendo el README, y usar cada endpoint documentado en Swagger obteniendo respuestas consistentes y bien validadas, sin necesitar explicacion adicional.

## Errores comunes a evitar

- Dejar `synchronize: true` activo en vez de migrations versionadas.
- Devolver la entidad completa sin DTO de respuesta, filtrando campos que no deberian exponerse.
- No validar los parametros de paginacion, permitiendo un `limit` sin tope.
- Omitir tests, incluso en un proyecto de practica: es donde mas facil es formar el habito.

## Extension opcional

Agregar autenticacion basica (ver proyecto siguiente) para proteger las operaciones de escritura, como puente hacia el proximo proyecto del capitulo.
