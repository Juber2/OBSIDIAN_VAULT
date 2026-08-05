---
tags:
  - PostgreSQL
  - Senior
  - Postgres-Senior
Creado: 2026-07-29
Relacionado:
  - Senior PostgreSQL
  - POSTGRESQL
---

# 03 - Migraciones Zero-Downtime

## Que vas a aprender

En este capitulo vas a aprender tecnicas para aplicar migraciones de esquema (ya vistas en profundidad en el capitulo de Postgres con NestJS) sin causar downtime en produccion, entendiendo que operaciones DDL bloquean tablas y como evitarlo.

## Explicacion

Ciertas operaciones DDL en Postgres requieren un **lock exclusivo** de la tabla durante toda su ejecucion (ver capitulo de Locks), bloqueando lecturas y escrituras: agregar una columna con un `DEFAULT` calculado (en versiones antiguas de Postgres, aunque desde la version 11 esto mejoro significativamente para valores constantes), crear un indice normal (`CREATE INDEX` sin `CONCURRENTLY`), o agregar un `NOT NULL` a una columna existente sin validacion previa. En una tabla grande, cualquiera de estas operaciones puede tomar minutos, bloqueando la aplicacion completa durante ese tiempo.

La tecnica de **zero-downtime migrations** evita estos locks largos: usar `CREATE INDEX CONCURRENTLY` (ya mencionado en el capitulo de Vistas Materializadas para refresh, el mismo principio aplica a indices) en vez de la version normal, agregar columnas nuevas como **nullable** primero y poblarlas en lotes pequeños despues (en vez de un `UPDATE` masivo bloqueante), y usar `ALTER TABLE ... ADD CONSTRAINT ... NOT VALID` seguido de `VALIDATE CONSTRAINT` por separado (que no bloquea escrituras mientras valida) para agregar constraints sin downtime.

## Modelo mental

Pensa una migracion tradicional bloqueante como cerrar completamente una tienda para hacer un cambio en el layout de estanterias, sin poder atender clientes mientras dura la obra. Una migracion zero-downtime es como hacer ese mismo cambio de a poco, estanteria por estanteria, en horarios de bajo trafico, manteniendo la tienda abierta y funcionando durante todo el proceso, sin que los clientes noten interrupcion alguna.

## Ejemplo base

```sql
-- Mal: bloquea la tabla completa durante toda la creacion del indice
CREATE INDEX idx_pedidos_estado ON pedidos (estado);

-- Bien: no bloquea escrituras, aunque tarda mas en completarse
CREATE INDEX CONCURRENTLY idx_pedidos_estado ON pedidos (estado);

-- Agregar un constraint sin bloquear escrituras durante la validacion
ALTER TABLE pedidos ADD CONSTRAINT chk_total_positivo CHECK (total > 0) NOT VALID;
-- Este paso valida los datos existentes SIN bloquear escrituras concurrentes
ALTER TABLE pedidos VALIDATE CONSTRAINT chk_total_positivo;

-- Poblar una columna nueva en lotes pequeños, en vez de un UPDATE masivo bloqueante
UPDATE pedidos SET nueva_columna = valor_calculado
WHERE id BETWEEN 1 AND 10000;
-- repetir en lotes hasta cubrir toda la tabla
```

## Como se usa en un proyecto real

1. Se usa `CREATE INDEX CONCURRENTLY` por default en cualquier tabla de produccion con trafico activo, aceptando el tiempo adicional que toma a cambio de no bloquear la aplicacion.
2. Se agrega cualquier constraint nuevo con `NOT VALID` + `VALIDATE CONSTRAINT` separado en tablas grandes de produccion, en vez de un `ADD CONSTRAINT` directo bloqueante.
3. Se poblan columnas nuevas en lotes pequeños con pausas entre cada lote, en vez de un `UPDATE` masivo que bloquearia la tabla durante todo el proceso.
4. Se planifica cualquier migracion de esquema significativa considerando el tamaño real de la tabla afectada, aplicando tecnicas zero-downtime proporcionalmente a ese riesgo.
5. Se conecta esto con el capitulo de Migrations del libro de NestJS, escribiendo SQL crudo dentro de las migrations de TypeORM cuando se necesitan estas tecnicas especificas de Postgres.

## Criterio junior

Un junior debe entender que ciertas operaciones DDL bloquean tablas completas, y conocer `CONCURRENTLY` como alternativa para indices.

## Criterio mid

Un mid debe aplicar el patron `NOT VALID` + `VALIDATE CONSTRAINT` para agregar constraints sin downtime en tablas grandes.

## Criterio senior

Un senior debe diseñar el proceso de migraciones zero-downtime del equipo, estableciendo cuando estas tecnicas son obligatorias segun el tamaño de tabla afectada, y revisando migraciones criticas en code review con este criterio.

## Errores comunes

- Usar `CREATE INDEX` normal en una tabla grande de produccion, causando un bloqueo de minutos que afecta a todos los usuarios activos.
- Agregar un constraint con `ADD CONSTRAINT` directo en una tabla grande, en vez del patron `NOT VALID` + `VALIDATE CONSTRAINT`.
- Ejecutar un `UPDATE` masivo para poblar una columna nueva en una tabla grande, bloqueando escrituras durante todo el proceso.
- No considerar el tamaño real de la tabla afectada antes de decidir si una migracion necesita tecnicas zero-downtime.

## Practica

- [ ] Crea un indice con `CONCURRENTLY` y compara conceptualmente el comportamiento con la version normal.
- [ ] Agrega un constraint usando el patron `NOT VALID` + `VALIDATE CONSTRAINT`.
- [ ] Diseña como poblarias una columna nueva en lotes pequeños para una tabla grande hipotetica.
- [ ] Escribe una migration de TypeORM que use SQL crudo con estas tecnicas zero-downtime.
- [ ] Documenta el criterio de tu equipo sobre cuando una migracion requiere tecnicas zero-downtime segun el tamaño de la tabla.

## Preguntas de repaso

- Por que `CREATE INDEX` normal puede causar downtime en una tabla grande de produccion?
- Que ventaja da el patron `NOT VALID` + `VALIDATE CONSTRAINT` sobre agregar un constraint directamente?
- Por que poblar una columna nueva en lotes pequeños es mejor que un `UPDATE` masivo en una tabla grande?
- Como decidirias si una migracion especifica necesita tecnicas zero-downtime?
