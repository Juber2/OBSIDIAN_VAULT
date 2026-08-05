---
tags:
  - PostgreSQL
  - Transacciones
  - Postgres-Transacciones
Creado: 2026-07-29
Relacionado:
  - Transacciones y Concurrencia
  - POSTGRESQL
---

# 02 - MVCC en Profundidad

## Que vas a aprender

En este capitulo vas a profundizar en **MVCC** (Multi-Version Concurrency Control), ya introducido en el capitulo de Arquitectura: como Postgres mantiene multiples versiones de cada fila, y por que esto genera la necesidad de `VACUUM` (profundizado en el capitulo de Administracion).

## Explicacion

Cuando actualizas o eliminas una fila en Postgres, el motor **no sobreescribe** el dato original inmediatamente: crea una **nueva version** de la fila (para `UPDATE`) o marca la version actual como invalida para transacciones futuras (para `DELETE`), manteniendo la version vieja disponible para transacciones que ya estaban en curso y necesitan ver el estado "de antes". Cada fila tiene metadata interna (`xmin`, `xmax`) que indica en que transaccion fue creada y en cual fue invalidada, permitiendo a Postgres determinar que version de una fila es visible para cada transaccion segun su momento de inicio.

El costo de este enfoque es que las versiones viejas de filas (**tuplas muertas**) se acumulan y ocupan espacio hasta que `VACUUM` las limpia definitivamente, una vez que ninguna transaccion activa las necesita ya. Sin `VACUUM` regular (normalmente automatico via `autovacuum`), una tabla con muchas actualizaciones puede hincharse significativamente en tamaño (**table bloat**), degradando performance.

## Modelo mental

Pensa MVCC como un sistema de control de versiones de documentos (como Git): cuando editas un documento, no se destruye la version anterior inmediatamente, se crea una version nueva, y cualquiera que ya estaba leyendo la version anterior sigue viendola intacta hasta que termine. Las versiones viejas eventualmente se limpian (`VACUUM`), pero mientras tanto ocupan espacio como "commits" historicos que ya nadie necesita consultar activamente.

## Ejemplo base

```sql
SELECT xmin, xmax, * FROM pedidos WHERE id = 1;
-- xmin: ID de transaccion que creo esta version
-- xmax: ID de transaccion que la invalido (0 si sigue siendo la version actual)

-- Ver tuplas muertas acumuladas en una tabla
SELECT relname, n_dead_tup, n_live_tup FROM pg_stat_user_tables WHERE relname = 'pedidos';

VACUUM pedidos;    -- limpia manualmente tuplas muertas (autovacuum lo hace automaticamente)
```

## Como se usa en un proyecto real

1. Se entiende que un `UPDATE` en Postgres, internamente, es mas parecido a un `DELETE` + `INSERT` de una version nueva que a una modificacion in-place, relevante para entender el costo real de actualizaciones frecuentes.
2. Se monitorea `n_dead_tup` en tablas con alta tasa de actualizacion, verificando que `autovacuum` esta limpiando tuplas muertas efectivamente.
3. Se ajusta la configuracion de `autovacuum` (ver capitulo de Administracion) en tablas de muy alto volumen de escritura donde el default podria no ser suficientemente agresivo.
4. Se conecta esto directamente con los niveles de aislamiento (tema siguiente), donde MVCC es el mecanismo tecnico que hace posible que lectores nunca bloqueen a escritores en Postgres.
5. Se investiga table bloat como causa raiz cuando una tabla con muchas actualizaciones crece en tamaño de forma desproporcionada respecto a su cantidad real de filas vivas.

## Criterio junior

Un junior debe entender conceptualmente que un `UPDATE` crea una nueva version de la fila en vez de modificarla in-place.

## Criterio mid

Un mid debe monitorear `n_dead_tup` de tablas con alta escritura, y entender la relacion entre MVCC y la necesidad de `VACUUM`.

## Criterio senior

Un senior debe ajustar la configuracion de `autovacuum` para tablas de alto volumen, y diagnosticar y resolver table bloat en produccion.

## Errores comunes

- No entender por que una tabla con muchas actualizaciones crece en tamaño en disco de forma desproporcionada a su cantidad real de filas vivas (table bloat).
- No monitorear `n_dead_tup`, perdiendo visibilidad de si `autovacuum` esta funcionando efectivamente en tablas criticas.
- Deshabilitar o configurar `autovacuum` de forma demasiado conservadora en tablas de alto volumen de escritura, permitiendo acumulacion excesiva de tuplas muertas.
- Confundir el concepto de MVCC con locking tradicional, esperando que lecturas bloqueen escrituras (o viceversa) como en otros modelos de concurrencia mas simples.

## Practica

- [ ] Consulta `xmin`/`xmax` de una fila antes y despues de un `UPDATE`.
- [ ] Revisa `n_dead_tup` de una tabla con actualizaciones frecuentes.
- [ ] Ejecuta `VACUUM` manualmente y observa el cambio en tuplas muertas.
- [ ] Investiga como identificarias table bloat en una tabla real de produccion.
- [ ] Explica con tus palabras por que MVCC permite que lectores nunca bloqueen a escritores.

## Preguntas de repaso

- Que hace Postgres internamente cuando ejecutas un `UPDATE`, en terminos de versiones de fila?
- Que son las tuplas muertas y por que se acumulan?
- Que rol cumple `VACUUM` respecto a las tuplas muertas?
- Como se relaciona MVCC con la capacidad de Postgres de que lectores nunca bloqueen a escritores?
