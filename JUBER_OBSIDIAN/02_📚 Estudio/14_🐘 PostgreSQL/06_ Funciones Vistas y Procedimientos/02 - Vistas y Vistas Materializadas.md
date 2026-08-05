---
tags:
  - PostgreSQL
  - Funciones
  - Postgres-Funciones
Creado: 2026-07-29
Relacionado:
  - Funciones Vistas y Procedimientos
  - POSTGRESQL
---

# 02 - Vistas y Vistas Materializadas

## Que vas a aprender

En este capitulo vas a aprender la diferencia entre **vistas** normales (consultas guardadas, siempre actualizadas) y **vistas materializadas** (resultado precalculado y almacenado fisicamente, que necesita refrescarse), y cuando usar cada una.

## Explicacion

Una **vista** (`CREATE VIEW`) es esencialmente una consulta SQL guardada con un nombre: cada vez que se consulta la vista, Postgres ejecuta la consulta subyacente en tiempo real, siempre mostrando datos actualizados al instante, pero sin ningun ahorro de tiempo de computo respecto a ejecutar la consulta original directamente. Son utiles principalmente para **simplificar** consultas complejas repetidas y para **encapsular** logica de acceso a datos (mostrando una interfaz simple sobre un join complejo), no para mejorar performance.

Una **vista materializada** (`CREATE MATERIALIZED VIEW`) si almacena fisicamente el resultado de la consulta en disco, dando lecturas muy rapidas, pero **no se actualiza automaticamente**: hay que refrescarla explicitamente (`REFRESH MATERIALIZED VIEW`), tipicamente en un horario programado (cron, ver capitulo de Scripting del libro de Linux) o disparada por un evento especifico, aceptando el tradeoff de mostrar datos potencialmente desactualizados a cambio de performance de lectura muy superior para reportes o dashboards costosos de calcular en tiempo real.

## Modelo mental

Pensa una vista normal como una formula guardada en una hoja de calculo: cada vez que la mirás, se recalcula en el momento con los datos actuales. Una vista materializada es como una fotografia impresa de esa hoja de calculo en un momento especifico: se ve instantaneamente sin recalcular nada, pero no refleja cambios posteriores hasta que alguien saque una foto nueva (`REFRESH`).

## Ejemplo base

```sql
CREATE VIEW pedidos_con_cliente AS
SELECT p.id, p.total, c.nombre AS cliente_nombre, c.email
FROM pedidos p
JOIN clientes c ON c.id = p.cliente_id;

SELECT * FROM pedidos_con_cliente WHERE cliente_nombre = 'Juber';
-- siempre datos actuales, pero recalcula el JOIN cada vez

CREATE MATERIALIZED VIEW resumen_ventas_mensual AS
SELECT date_trunc('month', creado_en) AS mes, SUM(total) AS total_mensual
FROM pedidos
GROUP BY date_trunc('month', creado_en);

REFRESH MATERIALIZED VIEW resumen_ventas_mensual;   -- actualiza el snapshot
REFRESH MATERIALIZED VIEW CONCURRENTLY resumen_ventas_mensual;   -- sin bloquear lecturas durante el refresh
```

## Como se usa en un proyecto real

1. Se usan vistas normales para simplificar consultas complejas repetidas y encapsular logica de acceso a datos, sin expectativa de mejora de performance.
2. Se usan vistas materializadas para dashboards o reportes costosos de calcular en tiempo real, aceptando el tradeoff de datos con cierto desfase controlado.
3. Se programa el `REFRESH` de vistas materializadas segun la frecuencia real que el negocio necesita (cada hora, cada dia), no mas frecuente de lo necesario.
4. Se usa `REFRESH MATERIALIZED VIEW CONCURRENTLY` (requiere un indice unico en la vista) para evitar bloquear lecturas mientras se refresca, relevante en sistemas de alto trafico.
5. Se documenta claramente cuando una vista materializada puede mostrar datos desactualizados, comunicando ese desfase a quien consume esos reportes.

## Criterio junior

Un junior debe entender la diferencia entre una vista normal (siempre actualizada, sin ahorro de computo) y una materializada (rapida, pero requiere refresh manual).

## Criterio mid

Un mid debe decidir cuando usar cada tipo de vista segun el caso de uso, y programar el refresh de vistas materializadas apropiadamente.

## Criterio senior

Un senior debe diseñar la estrategia de reportes y dashboards de un sistema usando vistas materializadas donde corresponda, balanceando frescura de datos contra costo de computo, y usando `CONCURRENTLY` para minimizar impacto en produccion.

## Errores comunes

- Esperar que una vista materializada refleje datos actualizados automaticamente sin haber ejecutado `REFRESH`.
- Usar una vista normal esperando mejora de performance, sin entender que simplemente ejecuta la misma consulta subyacente cada vez.
- No usar `CONCURRENTLY` al refrescar una vista materializada de alto trafico, bloqueando lecturas innecesariamente durante el refresh.
- No comunicar a los consumidores de un reporte que una vista materializada puede mostrar datos con cierto desfase temporal.

## Practica

- [ ] Crea una vista normal que simplifique un join complejo de tu esquema de practica.
- [ ] Crea una vista materializada para un reporte agregado costoso.
- [ ] Refresca la vista materializada y verifica el cambio de datos.
- [ ] Programa el refresh de esa vista materializada como una tarea de cron (ver libro de Linux).
- [ ] Documenta el desfase de datos aceptable de esa vista materializada para tu equipo.

## Preguntas de repaso

- Que diferencia hay entre una vista normal y una vista materializada en cuanto a performance y frescura de datos?
- Por que una vista normal no mejora la performance de una consulta compleja repetida?
- Que ventaja da `REFRESH MATERIALIZED VIEW CONCURRENTLY` sobre un refresh normal?
- Cuando elegirias una vista materializada sobre calcular un reporte en tiempo real?
