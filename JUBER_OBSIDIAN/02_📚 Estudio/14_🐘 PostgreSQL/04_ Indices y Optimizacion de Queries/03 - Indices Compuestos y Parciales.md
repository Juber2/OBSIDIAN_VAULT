---
tags:
  - PostgreSQL
  - Performance
  - Postgres-Indices
Creado: 2026-07-29
Relacionado:
  - Indices y Optimizacion de Queries
  - POSTGRESQL
---

# 03 - Indices Compuestos y Parciales

## Que vas a aprender

En este capitulo vas a aprender tecnicas avanzadas de indexacion: **indices compuestos** (multiples columnas) y su **orden critico**, e **indices parciales** (que solo cubren un subconjunto de filas), tecnicas que reducen significativamente el tamaño y aumentan la eficiencia de los indices.

## Explicacion

Un **indice compuesto** (`CREATE INDEX ON tabla (col1, col2)`) es util cuando las consultas filtran u ordenan por multiples columnas juntas, pero el **orden de las columnas importa muchisimo**: un indice en `(cliente_id, creado_en)` es eficiente para "filtrar por cliente_id, ordenar por creado_en" pero no ayuda a una consulta que solo filtra por `creado_en` sin `cliente_id` (el indice se puede usar de izquierda a derecha, no salteando la primera columna). La regla practica es poner primero la columna usada en igualdad exacta, despues la usada en rango u orden.

Un **indice parcial** (`CREATE INDEX ... WHERE condicion`) indexa solo las filas que cumplen una condicion, siendo mucho mas chico y eficiente cuando las consultas siempre filtran por esa misma condicion (por ejemplo, un indice solo sobre pedidos `WHERE estado = 'pendiente'`, si esa es la consulta mas frecuente y esos son pocos comparados con el total historico).

## Modelo mental

Pensa un indice compuesto como un directorio telefonico ordenado primero por apellido y despues por nombre: es muy eficiente para buscar "Garcia, Juan", pero inutil si buscas solo por nombre "Juan" sin apellido, porque el directorio no esta organizado de esa forma. Un indice parcial es como tener una libreta de direcciones mas chica que solo incluye contactos "activos", mucho mas rapida de hojear que la libreta completa cuando raramente necesitas buscar entre los contactos inactivos.

## Ejemplo base

```sql
-- Indice compuesto: eficiente para filtrar por cliente_id y ordenar por fecha
CREATE INDEX idx_pedidos_cliente_fecha ON pedidos (cliente_id, creado_en DESC);

-- Este SI aprovecha el indice completo:
SELECT * FROM pedidos WHERE cliente_id = 42 ORDER BY creado_en DESC;

-- Este SOLO aprovecha la primera columna del indice (cliente_id):
SELECT * FROM pedidos WHERE cliente_id = 42;

-- Este NO aprovecha el indice (salta la primera columna):
SELECT * FROM pedidos ORDER BY creado_en DESC;

-- Indice parcial: solo indexa pedidos pendientes, mucho mas chico
CREATE INDEX idx_pedidos_pendientes ON pedidos (creado_en) WHERE estado = 'pendiente';
```

## Como se usa en un proyecto real

1. Se diseñan indices compuestos analizando el patron de consulta real (que columnas se filtran juntas, en que orden), no agregando columnas al azar.
2. Se pone primero en un indice compuesto la columna filtrada por igualdad exacta, y despues la usada para rango u ordenamiento, siguiendo la regla practica de eficiencia.
3. Se usan indices parciales cuando una consulta especifica y frecuente siempre filtra por una condicion fija, reduciendo significativamente el tamaño del indice frente a indexar la tabla completa.
4. Se verifica con `EXPLAIN ANALYZE` (tema anterior) que un indice compuesto realmente se usa como se espera para las consultas objetivo.
5. Se documenta el proposito de cada indice compuesto o parcial no obvio, dado que su eficiencia depende de un patron de consulta especifico que puede no ser evidente al leer solo la definicion.

## Criterio junior

Un junior debe poder crear un indice compuesto basico y entender que el orden de las columnas afecta su utilidad.

## Criterio mid

Un mid debe diseñar indices compuestos analizando el patron real de consultas, y usar indices parciales cuando corresponda para reducir tamaño.

## Criterio senior

Un senior debe auditar la estrategia de indexacion completa de un sistema, eliminando indices redundantes o mal diseñados, y optimizando indices compuestos/parciales segun patrones de consulta medidos en produccion.

## Errores comunes

- Crear un indice compuesto con el orden de columnas equivocado, sin obtener el beneficio esperado para el patron de consulta real.
- No considerar indices parciales cuando una consulta frecuente siempre filtra por la misma condicion, indexando innecesariamente toda la tabla.
- Crear multiples indices redundantes que cubren casos superpuestos, aumentando el costo de escritura sin beneficio adicional de lectura.
- No verificar con `EXPLAIN ANALYZE` que un indice compuesto realmente se usa como se diseño, asumiendo su efectividad sin confirmarla.

## Practica

- [ ] Crea un indice compuesto en dos columnas y verifica con `EXPLAIN ANALYZE` que se usa correctamente para un filtro con ambas columnas.
- [ ] Verifica que ese mismo indice no ayuda a una consulta que solo filtra por la segunda columna.
- [ ] Crea un indice parcial para una condicion frecuente y compara su tamaño con un indice completo equivalente.
- [ ] Diseña el orden correcto de columnas para un indice compuesto segun un patron de consulta real de tu proyecto.
- [ ] Audita los indices de una tabla real buscando redundancia o mal diseño.

## Preguntas de repaso

- Por que el orden de columnas en un indice compuesto afecta que consultas pueden aprovecharlo?
- Cuando conviene un indice parcial sobre un indice completo de la tabla?
- Que regla practica seguirias para ordenar las columnas de un indice compuesto?
- Como verificarias que un indice compuesto realmente se esta usando para las consultas que lo motivaron?
