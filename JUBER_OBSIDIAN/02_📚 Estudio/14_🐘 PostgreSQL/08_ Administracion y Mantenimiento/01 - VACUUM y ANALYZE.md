---
tags:
  - PostgreSQL
  - Administracion
  - Postgres-Administracion
Creado: 2026-07-29
Relacionado:
  - Administracion y Mantenimiento
  - POSTGRESQL
---

# 01 - VACUUM y ANALYZE

## Que vas a aprender

En este capitulo vas a profundizar en `VACUUM` (ya introducido en el capitulo de MVCC) y `ANALYZE` (ya introducido en el capitulo de Query Planner): como funciona `autovacuum` en detalle, y como ajustar su configuracion para tablas de alto volumen.

## Explicacion

`VACUUM` limpia tuplas muertas (ver capitulo de MVCC) liberando espacio reutilizable dentro de la tabla (aunque `VACUUM` simple no devuelve ese espacio al sistema operativo; `VACUUM FULL` si lo hace, pero requiere un lock exclusivo de la tabla durante toda su ejecucion, por lo que se usa con mucha cautela en produccion). `ANALYZE` actualiza las estadisticas que el query planner usa para tomar decisiones.

**`autovacuum`** es un proceso en background que ejecuta ambas operaciones automaticamente cuando una tabla acumula suficientes cambios, configurable globalmente (`postgresql.conf`) o por tabla individual (`ALTER TABLE ... SET (autovacuum_vacuum_scale_factor = ...)`). En tablas de muy alto volumen de escritura, el default de `autovacuum` puede ser demasiado conservador (esperando demasiado tiempo entre ejecuciones), permitiendo acumulacion excesiva de tuplas muertas; ajustar sus parametros para que actue mas frecuentemente en esas tablas especificas es una tarea comun de administracion senior.

## Modelo mental

Pensa `autovacuum` como un servicio de limpieza automatico que pasa periodicamente por cada oficina (tabla) del edificio, pero que decide cuando limpiar cada una segun cuanto desorden (tuplas muertas) detecto la ultima vez. Si una oficina genera basura mucho mas rapido que el resto, puede necesitar un cronograma de limpieza mas frecuente especificamente configurado para ella, en vez de esperar al ciclo estandar de todo el edificio.

## Ejemplo base

```sql
VACUUM ANALYZE pedidos;    -- limpia tuplas muertas Y actualiza estadisticas en un solo comando

VACUUM FULL pedidos;        -- limpieza completa, devuelve espacio al SO, pero bloquea la tabla

SELECT relname, last_vacuum, last_autovacuum, last_analyze, last_autoanalyze
FROM pg_stat_user_tables WHERE relname = 'pedidos';

-- Ajustar autovacuum para una tabla especifica de alto volumen
ALTER TABLE pedidos SET (autovacuum_vacuum_scale_factor = 0.05);  -- mas agresivo que el default 0.2
```

## Como se usa en un proyecto real

1. Se confia en `autovacuum` como comportamiento default para la mayoria de tablas, sin necesitar intervencion manual constante.
2. Se ajusta `autovacuum_vacuum_scale_factor` (mas bajo = mas frecuente) para tablas de muy alto volumen de escritura donde el default resulta insuficiente.
3. Se evita `VACUUM FULL` en produccion durante horarios de trafico alto, dado que bloquea completamente la tabla durante su ejecucion; se reserva para ventanas de mantenimiento planificadas.
4. Se monitorea `pg_stat_user_tables` para verificar que `autovacuum` esta ejecutandose regularmente en tablas criticas, detectando configuraciones insuficientes antes de que se vuelvan un problema de performance.
5. Se conecta esto con el capitulo de MVCC, entendiendo que `VACUUM` es la contraparte necesaria del modelo de versiones multiples que hace posible la concurrencia sin bloqueos de lectura.

## Criterio junior

Un junior debe entender que `autovacuum` corre automaticamente y poder ejecutar `VACUUM ANALYZE` manualmente cuando corresponda.

## Criterio mid

Un mid debe monitorear `pg_stat_user_tables` para verificar la salud de `autovacuum`, y ejecutar `VACUUM ANALYZE` manual despues de cargas masivas de datos.

## Criterio senior

Un senior debe ajustar la configuracion de `autovacuum` por tabla segun patrones reales de escritura, y planificar ventanas de mantenimiento para operaciones como `VACUUM FULL` cuando sean genuinamente necesarias.

## Errores comunes

- No monitorear si `autovacuum` esta manteniendose al dia con tablas de alto volumen, descubriendo table bloat solo cuando ya afecta performance visiblemente.
- Ejecutar `VACUUM FULL` en horario de trafico alto sin considerar el lock exclusivo que genera, causando una interrupcion de servicio evitable.
- No ejecutar `ANALYZE` manual despues de una carga masiva de datos, dejando que el planner opere con estadisticas obsoletas hasta el proximo ciclo automatico.
- Deshabilitar `autovacuum` globalmente "para mejorar performance de escritura" sin entender el costo a largo plazo en acumulacion de tuplas muertas.

## Practica

- [ ] Ejecuta `VACUUM ANALYZE` manual en una tabla y revisa `pg_stat_user_tables` para confirmar el resultado.
- [ ] Simula una carga masiva de datos y verifica si `autovacuum` responde con la frecuencia esperada.
- [ ] Ajusta `autovacuum_vacuum_scale_factor` para una tabla especifica y explica el efecto esperado.
- [ ] Investiga las diferencias practicas y riesgos entre `VACUUM` simple y `VACUUM FULL`.
- [ ] Documenta un proceso de monitoreo regular de la salud de `autovacuum` para tablas criticas de tu proyecto.

## Preguntas de repaso

- Que diferencia hay entre `VACUUM` simple y `VACUUM FULL`?
- Por que `autovacuum` podria necesitar configuracion mas agresiva en una tabla de alto volumen de escritura?
- Que riesgo tiene ejecutar `VACUUM FULL` durante horario de trafico alto en produccion?
- Como verificarias la salud de `autovacuum` en una tabla critica?
