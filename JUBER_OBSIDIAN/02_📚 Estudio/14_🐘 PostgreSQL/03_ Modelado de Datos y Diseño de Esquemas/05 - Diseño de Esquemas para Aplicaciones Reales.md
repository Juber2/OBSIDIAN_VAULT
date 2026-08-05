---
tags:
  - PostgreSQL
  - Modelado
  - Postgres-Modelado
Creado: 2026-07-29
Relacionado:
  - Modelado de Datos y Diseño de Esquemas
  - POSTGRESQL
---

# 05 - Diseño de Esquemas para Aplicaciones Reales

## Que vas a aprender

En este capitulo vas a consolidar todo el capitulo diseñando un esquema completo para una aplicacion real: columnas de auditoria estandar, **soft deletes**, y el proceso practico de ir de un requisito de negocio a un esquema normalizado y bien restringido.

## Explicacion

Casi toda tabla de una aplicacion real se beneficia de **columnas de auditoria** estandar: `creado_en`, `actualizado_en` (con un trigger que la actualiza automaticamente, ver capitulo de Funciones y Triggers), y opcionalmente `creado_por` para trazabilidad de quien genero cada registro. **Soft delete** (marcar una fila como eliminada con una columna `eliminado_en timestamptz` en vez de borrarla realmente con `DELETE`) es un patron comun cuando se necesita preservar historial o permitir recuperacion, a costa de que toda consulta normal debe filtrar explicitamente `WHERE eliminado_en IS NULL`.

El proceso practico de diseño va de lo conceptual a lo concreto: identificar las **entidades** del dominio (sustantivos del negocio: usuario, pedido, producto), sus **atributos** (que datos tiene cada una), y sus **relaciones** (como se conectan entre si), aplicando despues normalizacion (tema 1), claves apropiadas (tema 2), constraints (tema 3), y el tipo de relacion correcto (tema 4) a cada pieza.

## Modelo mental

Pensa el diseño de un esquema completo como planificar los planos de un edificio nuevo: primero identificas que habitaciones necesita el edificio (entidades), que contiene cada habitacion (atributos), y como se conectan los pasillos entre ellas (relaciones), antes de empezar a poner ladrillos especificos (SQL). Las columnas de auditoria son como instalar camaras de seguridad estandar en cada habitacion desde el diseño inicial, no como una idea de ultimo momento.

## Ejemplo base

```sql
CREATE TABLE pedidos (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    cliente_id bigint NOT NULL REFERENCES clientes(id) ON DELETE RESTRICT,
    total numeric NOT NULL CHECK (total >= 0),
    estado text NOT NULL DEFAULT 'pendiente'
        CHECK (estado IN ('pendiente', 'pagado', 'enviado', 'cancelado')),
    creado_en timestamptz NOT NULL DEFAULT now(),
    actualizado_en timestamptz NOT NULL DEFAULT now(),
    eliminado_en timestamptz         -- NULL = activo, no NULL = soft-deleted
);

-- Toda consulta normal filtra soft deletes
SELECT * FROM pedidos WHERE eliminado_en IS NULL AND cliente_id = 5;
```

## Como se usa en un proyecto real

1. Se agregan columnas de auditoria (`creado_en`, `actualizado_en`) como estandar en toda tabla nueva de una aplicacion, sin excepcion salvo justificacion.
2. Se evalua soft delete solo donde realmente se necesita preservar historial o permitir recuperacion, dado que agrega complejidad (filtrar en cada consulta) que no siempre se justifica frente a un `DELETE` real respaldado por backups (ver capitulo de Administracion).
3. Se sigue el proceso de entidades -> atributos -> relaciones -> normalizacion -> claves -> constraints como flujo de diseño repetible para cualquier feature nueva.
4. Se conecta el esquema final con las entidades TypeORM del libro de NestJS, donde columnas de auditoria mapean a decoradores como `@CreateDateColumn()`/`@UpdateDateColumn()`.
5. Se documenta el esquema completo (diagrama entidad-relacion, decisiones de soft delete, convenciones de auditoria) como parte de la documentacion tecnica del proyecto.

## Criterio junior

Un junior debe poder agregar columnas de auditoria estandar a una tabla nueva y entender el concepto de soft delete.

## Criterio mid

Un mid debe decidir cuando soft delete se justifica versus un `DELETE` real respaldado por backups, y filtrar correctamente registros soft-deleted en todas las consultas relevantes.

## Criterio senior

Un senior debe definir el estandar de auditoria y soft delete para todo el equipo, y liderar el proceso de diseño de esquema desde requisitos de negocio hasta SQL final para features complejas nuevas.

## Errores comunes

- Olvidar filtrar registros soft-deleted en alguna consulta, mostrando datos que deberian estar "eliminados" al usuario.
- Usar soft delete en toda tabla por default sin evaluar si realmente se necesita, agregando complejidad innecesaria.
- No agregar columnas de auditoria desde el diseño inicial, teniendo que agregarlas despues con una migration mas compleja.
- Saltar directamente a escribir SQL sin pasar primero por el proceso conceptual de identificar entidades, atributos y relaciones.

## Practica

- [ ] Diseña un esquema completo para un dominio real (por ejemplo, un sistema de reservas) siguiendo el proceso completo: entidades, atributos, relaciones, normalizacion, claves, constraints.
- [ ] Agrega columnas de auditoria estandar a cada tabla del esquema.
- [ ] Decide para que tablas del esquema soft delete se justifica y para cuales no, documentando la razon.
- [ ] Escribe una consulta que filtre correctamente registros soft-deleted.
- [ ] Conecta el esquema completo con las entidades TypeORM equivalentes que escribirias en NestJS.

## Preguntas de repaso

- Que columnas de auditoria estandar agregarias a toda tabla nueva de una aplicacion?
- Cuando se justifica soft delete sobre un `DELETE` real respaldado por backups?
- Que riesgo tiene olvidar filtrar `eliminado_en IS NULL` en alguna consulta?
- Que proceso seguirias para ir de un requisito de negocio a un esquema SQL completo?
