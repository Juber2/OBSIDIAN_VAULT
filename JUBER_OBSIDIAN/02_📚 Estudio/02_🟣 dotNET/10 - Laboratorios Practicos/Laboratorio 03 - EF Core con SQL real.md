---
tags:
  - dotnet
  - laboratorio
  - efcore
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos dotNET
  - NET
---

# Laboratorio 03 - EF Core con SQL real

## Objetivo

Persistir la API usando EF Core con una base real, migraciones, indices, relaciones y queries optimizadas.

## Modelo

Users, Courses, Enrollments. User tiene email unico. Course tiene estado. Enrollment relaciona usuario y curso con fecha.

## Queries obligatorias

- Listado paginado con `AsNoTracking`.
- Detalle con proyeccion a DTO.
- Conteo de cursos por usuario.
- Busqueda por email normalizado.

## Revision SQL

Genera SQL o usa logging para revisar que no exista N+1 ni columnas innecesarias.

## Errores comunes

- Usar InMemory y creer que prueba SQL
- Include gigante
- Migracion sin revisar

## Practica

- [ ] Crear migration
- [ ] Revisar script SQL
- [ ] Agregar indice unico

## Relacionado

- [[Laboratorios Practicos dotNET]]
- [[NET]]
