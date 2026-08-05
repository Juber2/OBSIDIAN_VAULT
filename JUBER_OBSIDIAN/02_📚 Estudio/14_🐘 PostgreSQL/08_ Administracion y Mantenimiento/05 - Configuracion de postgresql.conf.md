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

# 05 - Configuracion de postgresql.conf

## Que vas a aprender

En este capitulo vas a cerrar el capitulo consolidando los parametros mas importantes de **`postgresql.conf`** para produccion: memoria (ya vistos conceptualmente en Arquitectura), checkpoints, logging, y como aplicar cambios de forma segura.

## Explicacion

Ademas de `shared_buffers`/`work_mem`/`max_connections` (ya vistos en el capitulo de Arquitectura), otros parametros criticos para produccion incluyen: `effective_cache_size` (una estimacion de cuanta memoria del sistema operativo esta disponible para cache de disco, usada por el planner para decidir entre Seq Scan e Index Scan, ver capitulo de Query Planner), `checkpoint_completion_target` (que tan gradualmente se escriben los checkpoints a disco, afectando picos de I/O), y `log_min_duration_statement` (loggea automaticamente cualquier query que tarde mas de un umbral especifico, la forma mas simple de detectar queries lentas sin necesitar revisar `pg_stat_statements` manualmente).

Algunos parametros requieren **reiniciar** el servidor completo para aplicarse (`shared_buffers`, `max_connections`), mientras que otros solo necesitan un **reload** (`work_mem`, `log_min_duration_statement`), sin interrumpir conexiones activas. Distinguir cuales es cual evita sorpresas al planificar cambios de configuracion en produccion.

## Modelo mental

Pensa `postgresql.conf` como el panel de control completo de una fabrica: algunos ajustes (como cambiar la velocidad de una cinta transportadora) se aplican de inmediato sin detener la produccion (reload), mientras que otros (como reconfigurar la potencia electrica general del edificio) requieren apagar y volver a encender toda la fabrica para tomar efecto (restart).

## Ejemplo base

```
# postgresql.conf (fragmentos clave)
shared_buffers = 4GB              # requiere restart
effective_cache_size = 12GB        # solo reload, ayuda al planner a estimar mejor
work_mem = 64MB                     # solo reload
checkpoint_completion_target = 0.9  # solo reload, checkpoints mas graduales
log_min_duration_statement = 500    # loggea queries que tardan mas de 500ms
```

```bash
sudo systemctl reload postgresql     # aplica cambios que solo necesitan reload
sudo systemctl restart postgresql    # aplica cambios que necesitan restart completo

SHOW ALL;    -- ver todos los parametros actuales
SELECT name, context FROM pg_settings WHERE name = 'shared_buffers';
-- "context" indica si necesita "postmaster" (restart) o "sighup" (reload)
```

## Como se usa en un proyecto real

1. Se configura `effective_cache_size` acorde a la memoria real disponible del servidor (tipicamente 50-75% de la RAM total), ayudando al planner a tomar mejores decisiones sobre cuando usar indices.
2. Se activa `log_min_duration_statement` con un umbral razonable (por ejemplo, 500ms) como red de seguridad basica para detectar queries lentas sin depender solo de revision manual de `pg_stat_statements`.
3. Se verifica con `pg_settings.context` si un cambio de configuracion necesita restart o solo reload, planificando ventanas de mantenimiento solo cuando realmente son necesarias.
4. Se documenta la configuracion de produccion como parte de la infraestructura como codigo (ver libro de Linux), versionada y reproducible.
5. Se conecta esto con el capitulo de Performance Avanzada, donde estos parametros se ajustan con mayor precision basandose en metricas reales medidas.

## Criterio junior

Un junior debe poder revisar valores actuales de configuracion con `SHOW` y aplicar un cambio simple con reload.

## Criterio mid

Un mid debe configurar `effective_cache_size` y `log_min_duration_statement` apropiadamente para un servidor de produccion.

## Criterio senior

Un senior debe diseñar la configuracion completa de `postgresql.conf` para servidores de produccion segun el hardware disponible y el patron de carga real, documentandola como infraestructura como codigo.

## Errores comunes

- No configurar `effective_cache_size`, dejando al planner con una estimacion pesimista por defecto que puede llevar a decisiones subóptimas de plan de ejecucion.
- No activar `log_min_duration_statement`, perdiendo la forma mas simple de detectar queries lentas automaticamente.
- Intentar aplicar con `reload` un parametro que en realidad requiere `restart`, sorprendiendose de que el cambio no tiene efecto.
- No documentar ni versionar la configuracion de produccion, perdiendo reproducibilidad ante un servidor nuevo.

## Practica

- [ ] Revisa los valores actuales de `shared_buffers`, `effective_cache_size` y `work_mem` en tu instalacion.
- [ ] Activa `log_min_duration_statement` con un umbral razonable y verifica que loggea queries lentas.
- [ ] Verifica con `pg_settings` si un parametro especifico requiere restart o solo reload.
- [ ] Ajusta `effective_cache_size` segun la memoria disponible de un servidor hipotetico.
- [ ] Documenta la configuracion completa de `postgresql.conf` que usarias para un servidor de produccion.

## Preguntas de repaso

- Que ayuda a estimar mejor `effective_cache_size` en las decisiones del query planner?
- Para que sirve `log_min_duration_statement` y por que es una red de seguridad simple pero efectiva?
- Como verificarias si un cambio de configuracion necesita restart o solo reload?
- Por que documentar y versionar la configuracion de produccion es importante para reproducibilidad?
