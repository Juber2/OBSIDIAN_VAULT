---
tags:
  - PostgreSQL
  - Proyectos
  - Postgres-Proyectos
Creado: 2026-07-29
Relacionado:
  - Proyectos
  - POSTGRESQL
---

# 04 - Proyecto API NestJS con Postgres Completo

## Que vas a construir

Una API NestJS completa conectada al esquema de los proyectos anteriores, aplicando de punta a punta el capitulo de Postgres con NestJS y TypeORM, cerrando el circulo entre ambos libros de este vault.

## Alcance funcional

- API NestJS con entidades TypeORM mapeando el esquema del primer proyecto, incluyendo al menos una columna JSONB y un ENUM de Postgres.
- Migrations de TypeORM para todo el esquema, con al menos una que use SQL crudo para una feature especifica de Postgres (indice GIN, o similar).
- Un repositorio custom con al menos un metodo usando QueryBuilder y otro con SQL crudo parametrizado.
- Tests de integracion con Postgres real en Docker (conectando con el libro de Docker), con limpieza de estado entre tests.
- Pool de conexiones dimensionado explicitamente, con SSL configurado condicionalmente para produccion.

## Checklist tecnica

- [ ] Las entidades TypeORM mapean correctamente tipos especificos de Postgres (JSONB, ENUM, array si aplica).
- [ ] Existe al menos una migration con SQL crudo documentado explicando por que era necesario.
- [ ] El repositorio custom tiene al menos un metodo con SQL crudo correctamente parametrizado.
- [ ] Los tests de integracion corren contra Postgres real en Docker, con limpieza de estado verificada.
- [ ] La configuracion de conexion incluye pool dimensionado y manejo de reintentos.

## Criterio de exito

El proyecto se considera completo cuando la API completa (levantada con Docker Compose) expone endpoints funcionales que ejercitan features especificas de Postgres (JSONB, indices GIN), con tests de integracion reales que pasan de forma consistente y una configuracion de conexion lista para produccion.

## Errores comunes a evitar

- No revisar el SQL autogenerado por las migrations antes de aplicarlo.
- Usar SQL crudo sin parametrizar correctamente, dejando un riesgo de inyeccion SQL.
- No limpiar el estado entre tests de integracion, generando fallos intermitentes dependientes del orden.

## Extension opcional

Agregar Row Level Security (ver capitulo Senior) a una tabla multi-tenant simulada, integrando el establecimiento del contexto de tenant desde un interceptor de NestJS.
