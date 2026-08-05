---
tags:
  - NestJS
  - TypeScript
  - Nest-APIs
Creado: 2026-07-27
Relacionado:
  - APIs REST NestJS
  - NESTJS
---

# 04 - Paginacion Filtros y Ordenamiento

## Que vas a aprender

En este capitulo vas a aprender a diseñar endpoints de listado robustos: **paginacion** (offset vs cursor), **filtros** dinamicos por query params, y **ordenamiento** configurable, sin sacrificar performance ni seguridad.

## Explicacion

La paginacion por **offset** (`page`, `limit`) es simple de implementar y entender, pero degrada en performance sobre tablas muy grandes (el motor de base de datos igual recorre las filas salteadas). La paginacion por **cursor** (basada en el ultimo ID/valor visto) escala mejor para datasets grandes y listados en tiempo real, a costa de mayor complejidad de implementacion.

Los filtros dinamicos deben construirse con una whitelist explicita de campos filtrables: nunca se debe permitir que el cliente arme una condicion SQL arbitraria a partir de un query param libre, porque eso abre la puerta a inyeccion o a queries extremadamente costosas.

## Modelo mental

Pensa un endpoint de listado como una vidriera con reglas: el cliente puede pedir "mostrame la pagina 3, ordenado por fecha, solo los activos", pero nunca puede pedir "ejecutame esta condicion arbitraria sobre la tabla". Vos decidis de antemano el menu de filtros y ordenamientos permitidos.

## Ejemplo base

```typescript
export class QueryUsersDto {
  @IsOptional() @IsInt() @Min(1) page = 1;
  @IsOptional() @IsInt() @Min(1) @Max(100) limit = 20;
  @IsOptional() @IsIn(['active', 'inactive']) status?: string;
  @IsOptional() @IsIn(['createdAt', 'email']) sortBy = 'createdAt';
  @IsOptional() @IsIn(['ASC', 'DESC']) order: 'ASC' | 'DESC' = 'DESC';
}

async findAll(query: QueryUsersDto) {
  const [items, total] = await this.usersRepo.findAndCount({
    where: query.status ? { status: query.status } : {},
    order: { [query.sortBy]: query.order },
    skip: (query.page - 1) * query.limit,
    take: query.limit,
  });
  return { items, total, page: query.page, limit: query.limit };
}
```

## Como se usa en un proyecto real

1. Se define un DTO de query con whitelist explicita de campos filtrables y ordenables.
2. Se limita el `limit` maximo permitido para evitar que un cliente pida miles de registros de una vez.
3. Se usa paginacion por cursor en endpoints de alto volumen o feeds en tiempo real, offset en listados administrativos comunes.
4. La respuesta de listado siempre incluye metadata (`total`, `page`, `hasNextPage`) para que el frontend pueda construir la UI de paginacion.
5. Se indexan en base de datos las columnas usadas para ordenar y filtrar frecuentemente.

## Criterio junior

Un junior debe poder implementar paginacion offset basica con `page` y `limit`, y aplicar un filtro simple validado.

## Criterio mid

Un mid debe implementar filtros dinamicos con whitelist segura, definir limites razonables de `limit`, e indexar correctamente las columnas usadas en filtros y ordenamiento.

## Criterio senior

Un senior debe decidir cuando offset ya no alcanza y migrar a paginacion por cursor, y definir la convencion de listados (formato de respuesta, nombres de query params) para toda la API del equipo.

## Errores comunes

- Permitir que el cliente pase cualquier nombre de columna como `sortBy` sin whitelist, arriesgando errores o exposicion de columnas internas.
- No limitar el `limit` maximo, permitiendo que un cliente pida datasets enormes en una sola request.
- Usar paginacion offset en un dataset muy grande y sufrir degradacion de performance a medida que crece la pagina pedida.
- No indexar las columnas usadas para filtrar y ordenar, generando full table scans.

## Practica

- [ ] Implementa un endpoint de listado con paginacion offset y filtros validados.
- [ ] Define una whitelist de campos ordenables y filtrables.
- [ ] Mide el tiempo de respuesta de una pagina "profunda" (offset alto) sobre una tabla grande.
- [ ] Investiga como se implementaria paginacion por cursor para el mismo endpoint.
- [ ] Anota cuando en tu proyecto conviene cursor sobre offset.

## Preguntas de repaso

- Por que la paginacion offset degrada en tablas muy grandes?
- Que riesgo hay en permitir un `sortBy` sin whitelist?
- Que ventaja tiene la paginacion por cursor sobre offset en un feed en tiempo real?
- Que metadata deberia incluir siempre una respuesta de listado paginado?
