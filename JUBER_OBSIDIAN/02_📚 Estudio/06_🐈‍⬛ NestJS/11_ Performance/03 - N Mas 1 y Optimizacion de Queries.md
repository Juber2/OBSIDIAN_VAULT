---
tags:
  - NestJS
  - TypeScript
  - Nest-Performance
Creado: 2026-07-27
Relacionado:
  - Performance NestJS
  - NESTJS
---

# 03 - N Mas 1 y Optimizacion de Queries

## Que vas a aprender

En este capitulo vas a profundizar en la optimizacion de queries de base de datos en NestJS: retomar y ampliar el problema de **N+1** (ya visto en el capitulo de relaciones TypeORM), indices, y como leer un plan de ejecucion (`EXPLAIN`) para diagnosticar queries lentas.

## Explicacion

Mas alla del N+1 clasico, la performance de queries depende de tres factores principales: **indices** correctos en las columnas usadas en `WHERE`, `JOIN` y `ORDER BY`; **seleccion de columnas** (traer solo lo necesario en vez de `SELECT *`); y **forma de la query** (evitar joins innecesarios, subqueries costosas cuando un join directo alcanza). El comando `EXPLAIN ANALYZE` de Postgres (o equivalente en MySQL) muestra el plan real que el motor de base de datos elige para ejecutar una query, revelando si esta usando un indice o haciendo un costoso "sequential scan" sobre toda la tabla.

## Modelo mental

Pensa el plan de ejecucion como el GPS de una query: te muestra la ruta exacta que el motor de base de datos eligio para llegar al resultado. Si esa ruta pasa por "revisar cada fila de la tabla" en vez de "ir directo con un indice", vas a notar el problema mucho antes en el plan que esperando a que la query sea lenta en produccion.

## Ejemplo base

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = '123' ORDER BY created_at DESC LIMIT 20;

-- Sin indice: Seq Scan on orders (costoso en tablas grandes)
-- Con indice en (user_id, created_at): Index Scan (mucho mas rapido)
```

```typescript
// TypeORM: seleccionar solo columnas necesarias en vez de la entidad completa
const orders = await this.ordersRepo.find({
  select: ['id', 'total', 'createdAt'],
  where: { userId },
  order: { createdAt: 'DESC' },
  take: 20,
});
```

## Como se usa en un proyecto real

1. Se revisan periodicamente las queries mas frecuentes de la aplicacion con `EXPLAIN ANALYZE` para verificar que usan indices.
2. Se crean indices compuestos que reflejen los patrones de consulta reales (filtro + orden usados juntos frecuentemente).
3. Se evita `SELECT *` (o el equivalente en TypeORM de traer la entidad completa) cuando el endpoint solo necesita un subconjunto de columnas.
4. Se monitorean queries lentas en produccion con herramientas de la base de datos (`pg_stat_statements` en Postgres) para priorizar optimizaciones donde realmente importan.
5. Se balancea: no se optimiza prematuramente cada query desde el dia uno, pero se revisan las que aparecen como lentas bajo carga real.

## Criterio junior

Un junior debe poder ejecutar `EXPLAIN` sobre una query simple e identificar si esta usando un indice o no.

## Criterio mid

Un mid debe crear indices compuestos apropiados segun los patrones de consulta reales, y optimizar queries evitando traer columnas o relaciones innecesarias.

## Criterio senior

Un senior debe establecer el proceso de monitoreo continuo de queries lentas en produccion, priorizar optimizaciones segun impacto real medido, y balancear el costo de mantener indices (que ralentizan escrituras) contra el beneficio en lecturas.

## Errores comunes

- No revisar el plan de ejecucion antes de asumir que una query es rapida solo porque "funciona" en desarrollo con pocos datos.
- Crear indices en exceso sin medir el impacto negativo en la performance de escritura.
- Traer columnas o relaciones completas cuando el endpoint solo necesita un subconjunto pequeño de datos.
- No monitorear queries lentas en produccion, descubriendo problemas de performance solo cuando un usuario se queja.

## Practica

- [ ] Ejecuta `EXPLAIN ANALYZE` sobre una query real de tu proyecto y analiza el plan.
- [ ] Crea un indice compuesto para un patron de consulta frecuente y mide la mejora.
- [ ] Refactoriza una query que trae columnas innecesarias para seleccionar solo lo requerido.
- [ ] Investiga como identificar las queries mas lentas en produccion con tu motor de base de datos.
- [ ] Anota una checklist propia de revision de performance antes de mergear una feature con queries nuevas.

## Preguntas de repaso

- Que diferencia hay entre un "Seq Scan" y un "Index Scan" en un plan de ejecucion?
- Por que crear indices en exceso puede ser contraproducente?
- Que ventaja tiene seleccionar solo las columnas necesarias en vez de traer la entidad completa?
- Como priorizarias que queries optimizar primero en un sistema en produccion?
