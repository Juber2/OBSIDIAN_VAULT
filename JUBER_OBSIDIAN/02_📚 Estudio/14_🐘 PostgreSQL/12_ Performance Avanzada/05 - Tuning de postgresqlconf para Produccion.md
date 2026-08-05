---
tags:
  - PostgreSQL
  - Performance
  - Postgres-Performance-Avanzada
Creado: 2026-07-29
Relacionado:
  - Performance Avanzada
  - POSTGRESQL
---

# 05 - Tuning de postgresql.conf para Produccion

## Que vas a aprender

En este capitulo vas a cerrar el capitulo consolidando el ajuste fino de `postgresql.conf` (ya visto en el capitulo de Administracion) especificamente basado en **datos medidos** con las herramientas de este capitulo, en vez de valores genericos copiados de internet.

## Explicacion

El tuning efectivo de Postgres no es aplicar una lista generica de "mejores valores" (esos valores dependen completamente del hardware especifico, patron de carga, y tipo de aplicacion), sino un proceso iterativo: medir con `pg_stat_statements` y `EXPLAIN ANALYZE` (ya vistos), identificar el cuello de botella real (memoria insuficiente causando I/O de disco excesivo, checkpoints demasiado agresivos causando picos de latencia, conexiones excesivas agotando `max_connections`), ajustar el parametro especifico relacionado, y volver a medir para confirmar la mejora real.

Herramientas como **PGTune** (una calculadora online/CLI que sugiere valores iniciales razonables segun el hardware disponible y el tipo de carga: OLTP transaccional, data warehouse, mixta) son un buen punto de **partida**, nunca un destino final: los valores sugeridos deben ajustarse despues con datos reales medidos del sistema especifico en produccion.

## Modelo mental

Pensa el tuning de Postgres como afinar un instrumento musical: una calculadora como PGTune te da una afinacion inicial razonable basada en el tipo de instrumento (hardware/carga), pero el afinador experto sigue escuchando (midiendo con las herramientas de este capitulo) y ajustando fino segun como suena realmente ese instrumento especifico en esa sala especifica, no solo confiando en el numero de fabrica.

## Ejemplo base

```
Proceso iterativo de tuning:

1. Medir: pg_stat_statements revela que hay muchas consultas con alta latencia de I/O
2. Diagnosticar: iostat (ver libro de Linux) confirma alto %util de disco
3. Hipotesis: shared_buffers/effective_cache_size insuficientes, forzando I/O de disco repetido
4. Ajustar: aumentar shared_buffers de 2GB a 4GB (25% de 16GB RAM disponible)
5. Volver a medir: comparar mean_exec_time de las mismas queries antes/despues del cambio
```

```sql
-- Verificar cache hit ratio: deberia ser >99% en un sistema bien dimensionado
SELECT sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) AS cache_hit_ratio
FROM pg_statio_user_tables;
```

## Como se usa en un proyecto real

1. Se usa PGTune como punto de partida razonable segun el hardware disponible y tipo de carga, nunca como configuracion final sin validacion.
2. Se aplica el proceso iterativo completo (medir, diagnosticar, ajustar, volver a medir) para cada cambio de configuracion significativo, con evidencia real en cada paso.
3. Se monitorea el cache hit ratio como metrica clave de si `shared_buffers`/`effective_cache_size` estan bien dimensionados para el volumen de datos actual.
4. Se coordina el tuning de Postgres con el dimensionamiento de recursos del servidor o container (ver capitulo de Costos del libro de Docker si corre containerizado), asegurando coherencia entre limites externos e internos.
5. Se documenta cada cambio de configuracion aplicado y su justificacion medida, construyendo un historial de decisiones de tuning para el equipo.

## Criterio junior

Un junior debe poder usar PGTune para obtener una configuracion inicial razonable segun el hardware disponible.

## Criterio mid

Un mid debe aplicar el proceso iterativo de medir-diagnosticar-ajustar-remedir para un cambio de configuracion especifico, verificando el cache hit ratio.

## Criterio senior

Un senior debe liderar el proceso de tuning continuo de servidores de produccion criticos, documentando decisiones basadas en evidencia medida, y coordinando con el dimensionamiento general de infraestructura.

## Errores comunes

- Copiar valores de configuracion genericos de internet sin considerar el hardware y carga especificos del sistema real.
- Usar PGTune como configuracion final sin validar despues con datos reales medidos del sistema en produccion.
- Ajustar multiples parametros simultaneamente sin medir el efecto de cada uno por separado, perdiendo claridad sobre que cambio realmente ayudo.
- No monitorear el cache hit ratio, perdiendo una señal directa de si la memoria asignada es suficiente para el volumen de datos actual.

## Practica

- [ ] Usa PGTune (o equivalente) para obtener una configuracion inicial para un servidor hipotetico.
- [ ] Verifica el cache hit ratio de un sistema real o de practica.
- [ ] Aplica el proceso iterativo completo para un ajuste especifico, midiendo antes y despues.
- [ ] Documenta un cambio de configuracion real aplicado, con su justificacion basada en evidencia.
- [ ] Coordina el tuning de Postgres con el dimensionamiento de recursos del contenedor o servidor donde corre.

## Preguntas de repaso

- Por que copiar valores de configuracion genericos de internet no es un enfoque confiable de tuning?
- Que rol cumple PGTune, y por que sus sugerencias no deberian ser la configuracion final sin validacion?
- Que indica un cache hit ratio bajo sobre el dimensionamiento de memoria del sistema?
- Como aplicarias el proceso iterativo completo de tuning a un cambio de configuracion especifico?

## Cierre del libro

Completar este capitulo cierra el recorrido completo de PostgreSQL desde fundamentos hasta produccion de alto rendimiento: SQL solido, modelado relacional correcto, indices y optimizacion basados en evidencia, transacciones y concurrencia con MVCC, programabilidad avanzada, tipos de datos que aprovechan lo mejor del motor, administracion y alta disponibilidad confiables, despliegue en la nube, integracion completa con NestJS/TypeORM, y ahora tuning de performance basado en datos reales medidos. El proximo paso natural es aplicar este conocimiento profundo directamente en tus proyectos reales, como [[NESTJS|el backend NestJS]] corriendo sobre [[DOCKER|Docker]] y [[LINUX|Linux]].
