---
tags:
  - SQLServer
  - Funciones
  - SQLServer-Funciones
Creado: 2026-07-29
Relacionado:
  - Funciones Vistas y Procedimientos
  - SQL SERVER
---

# 02 - Vistas e Indexed Views

## Que vas a aprender

En este capitulo vas a aprender vistas normales en T-SQL (identicas en proposito a Postgres) e **Indexed Views**, el equivalente de SQL Server a las vistas materializadas de Postgres (ya estudiadas en profundidad), con una diferencia clave de actualizacion.

## Explicacion

Una vista normal (`CREATE VIEW`) es identica en comportamiento a Postgres: una consulta guardada que se recalcula en tiempo real cada vez que se consulta. Una **Indexed View** (`CREATE VIEW ... WITH SCHEMABINDING` + `CREATE UNIQUE CLUSTERED INDEX` sobre esa vista) es el equivalente de una vista materializada, con una diferencia critica: mientras una vista materializada de Postgres necesita `REFRESH` manual explicito, una **Indexed View se actualiza automaticamente** en cada `INSERT`/`UPDATE`/`DELETE` de las tablas subyacentes, manteniendose siempre sincronizada, a costa de agregar overhead a cada escritura sobre esas tablas.

Esto representa un tradeoff distinto al ya conocido en Postgres: la vista materializada de Postgres es "barata de escribir, cara de leer hasta el proximo refresh" (datos potencialmente desactualizados), mientras que la Indexed View de SQL Server es "cara de escribir, siempre actualizada al leer" (sin desfase, pero con costo constante en cada escritura de las tablas base).

## Modelo mental

Pensa una vista materializada de Postgres como una fotografia impresa que hay que actualizar manualmente de vez en cuando. Una Indexed View de SQL Server es como un espejo magico que se actualiza automaticamente en tiempo real cada vez que el original cambia, sin necesitar que nadie tome una foto nueva, pero ese espejo mismo consume un poco de esfuerzo extra cada vez que el original se modifica, para mantenerse sincronizado instantaneamente.

## Ejemplo base

```sql
-- Vista normal, identica en comportamiento a Postgres
CREATE VIEW PedidosConCliente AS
SELECT p.Id, p.Total, c.Nombre AS ClienteNombre
FROM Pedidos p JOIN Clientes c ON c.Id = p.ClienteId;

-- Indexed View: requiere SCHEMABINDING y un indice clustered unico
CREATE VIEW ResumenVentasPorCliente
WITH SCHEMABINDING
AS
SELECT ClienteId, COUNT_BIG(*) AS CantidadPedidos, SUM(Total) AS TotalGastado
FROM dbo.Pedidos
GROUP BY ClienteId;

CREATE UNIQUE CLUSTERED INDEX IX_ResumenVentas ON ResumenVentasPorCliente (ClienteId);
-- A partir de aca, esta vista se mantiene actualizada automaticamente en cada escritura
```

## Como se usa en un proyecto real

1. Se usan vistas normales para simplificar consultas complejas repetidas, identico al criterio ya establecido en Postgres.
2. Se evaluan Indexed Views para reportes que necesitan estar **siempre actualizados** sin desfase, aceptando el costo constante en escrituras, a diferencia de una vista materializada que se refresca periodicamente.
3. Se requiere `WITH SCHEMABINDING`, lo cual **impide modificar el esquema** de las tablas subyacentes sin primero eliminar la Indexed View, una restriccion operacional a considerar.
4. Se compara conscientemente este tradeoff (siempre actualizado pero costoso en escritura) contra vistas materializadas de Postgres (barato en escritura pero con desfase controlado) al decidir la estrategia para un reporte especifico.
5. Se documenta claramente cuando una vista es Indexed, dado que su comportamiento de mantenimiento automatico no es evidente solo leyendo el nombre.

## Criterio junior

Un junior debe poder crear una vista normal en T-SQL, identica en proposito a las ya estudiadas en Postgres.

## Criterio mid

Un mid debe crear una Indexed View basica, entendiendo el requisito de `SCHEMABINDING` y el tradeoff de mantenimiento automatico.

## Criterio senior

Un senior debe decidir entre Indexed Views y refrescado periodico (equivalente conceptual a vistas materializadas de Postgres) segun el requisito real de frescura de datos versus costo de escritura.

## Errores comunes

- Confundir una Indexed View con una vista materializada de Postgres sin entender la diferencia clave de actualizacion automatica versus manual.
- Crear una Indexed View sobre una tabla con escrituras muy frecuentes sin considerar el overhead constante que agrega a cada operacion.
- No entender que `SCHEMABINDING` impide cambios de esquema sin eliminar primero la vista, complicando futuras migraciones.
- No documentar que una vista es Indexed, generando confusion sobre su comportamiento real de actualizacion.

## Practica

- [ ] Crea una vista normal simplificando un join complejo.
- [ ] Crea una Indexed View con `SCHEMABINDING` y un indice clustered unico.
- [ ] Verifica que la Indexed View se actualiza automaticamente al modificar las tablas subyacentes.
- [ ] Compara explicitamente el tradeoff de Indexed View versus vista materializada de Postgres.
- [ ] Identifica un reporte real donde una Indexed View seria apropiada versus uno donde una vista materializada periodica seria mejor.

## Preguntas de repaso

- Que diferencia clave hay entre una Indexed View de SQL Server y una vista materializada de Postgres?
- Que requisito impone `SCHEMABINDING` sobre las tablas subyacentes?
- Que tradeoff de costo/beneficio representa una Indexed View frente a una vista materializada refrescada periodicamente?
- Cuando elegirias una Indexed View sobre una vista normal simple?
