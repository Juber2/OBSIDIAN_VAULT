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

# 03 - Point-in-Time Recovery con WAL

## Que vas a aprender

En este capitulo vas a aprender **Point-in-Time Recovery (PITR)**: como el **WAL** (Write-Ahead Log, ya mencionado en el capitulo de ACID como base de la Durabilidad) permite restaurar una base de datos a un momento **exacto** en el tiempo, no solo al momento de un backup completo previo.

## Explicacion

Postgres registra **cada cambio** que hace a la base de datos en el WAL antes de aplicarlo realmente a los archivos de datos (de ahi "write-ahead"), garantizando durabilidad ante una caida. PITR aprovecha esto: se toma un **backup base** completo (fisico, con `pg_basebackup`) una vez, y despues se conservan **todos los archivos WAL** generados desde ese momento. Para restaurar a cualquier momento posterior al backup base, se restaura el backup base y se **reproducen** los archivos WAL secuencialmente hasta el punto exacto deseado (por timestamp, o hasta antes de una transaccion especifica).

Esto es fundamentalmente distinto de `pg_dump`: mientras `pg_dump` solo puede restaurar al momento exacto en que se genero, PITR permite restaurar a **cualquier segundo** entre el backup base y el presente, siendo la herramienta correcta para recuperarse de un error humano descubierto horas o dias despues de haber ocurrido (por ejemplo, un `DELETE` sin `WHERE`, ya visto en el capitulo de SQL Fundamentals, ejecutado por error a las 3pm, descubierto a las 6pm).

## Modelo mental

Pensa el WAL como una grabacion continua de video de todo lo que pasa en la base de datos, segundo a segundo. Un backup con `pg_dump` es como una unica fotografia tomada en un momento especifico. PITR es como tener la grabacion completa: podes "rebobinar" hasta exactamente el segundo antes de que algo malo pasara, en vez de tener que conformarte con la fotografia mas cercana disponible, que podria ser de horas antes del incidente.

## Ejemplo base

```bash
# Configurar archivado continuo de WAL en postgresql.conf
# archive_mode = on
# archive_command = 'cp %p /respaldo/wal_archive/%f'

pg_basebackup -D /respaldo/backup_base -Ft -z -P    # backup base fisico completo
```

```
# recovery.signal + postgresql.conf en el servidor de restauracion
restore_command = 'cp /respaldo/wal_archive/%f %p'
recovery_target_time = '2026-07-29 14:59:00'   # justo antes del DELETE accidental
```

## Como se usa en un proyecto real

1. Se configura archivado continuo de WAL en cualquier base de datos de produccion donde la capacidad de recuperacion a un punto exacto en el tiempo sea critica.
2. Se combina un `pg_basebackup` periodico (semanal, por ejemplo) con archivado continuo de WAL entre medio, minimizando cuantos WAL hay que reproducir para llegar a un punto reciente.
3. Se usa PITR especificamente para recuperarse de errores humanos descubiertos tarde (un `DELETE`/`UPDATE` sin `WHERE`, ver capitulo de SQL Fundamentals), restaurando a segundos antes del error.
4. Se prueba el proceso completo de PITR en un ambiente de prueba periodicamente, dado que es mas complejo de ejecutar correctamente que una restauracion simple de `pg_dump`.
5. Se documenta el procedimiento exacto de PITR paso a paso, dado que es una operacion de alto estres que se ejecuta raramente y bajo presion cuando realmente se necesita.

## Criterio junior

Un junior debe entender conceptualmente que PITR permite restaurar a cualquier momento en el tiempo, no solo al momento de un backup completo.

## Criterio mid

Un mid debe configurar archivado continuo de WAL y ejecutar un `pg_basebackup`, entendiendo los componentes necesarios para PITR.

## Criterio senior

Un senior debe diseñar e implementar la estrategia completa de PITR para sistemas criticos, probando el proceso regularmente y documentandolo para uso en situaciones de alto estres.

## Errores comunes

- Confiar solo en `pg_dump` periodico para un sistema donde un error humano descubierto tarde seria catastrofico, sin tener capacidad de PITR disponible.
- No probar nunca el proceso completo de PITR, descubriendo problemas de configuracion recien durante un incidente real de alta presion.
- No documentar el procedimiento exacto de PITR, dejando que solo una persona sepa ejecutarlo correctamente bajo presion.
- Configurar archivado de WAL sin verificar que el almacenamiento destino tiene capacidad suficiente para el volumen de WAL generado a largo plazo.

## Practica

- [ ] Configura archivado continuo de WAL en una instalacion de Postgres de practica.
- [ ] Genera un `pg_basebackup` completo.
- [ ] Simula un error (como un `DELETE` sin `WHERE` en una tabla de prueba) y practica restaurar con PITR a un momento justo antes de ese error.
- [ ] Documenta el procedimiento completo paso a paso que seguiste.
- [ ] Compara los casos de uso donde usarias `pg_dump` simple versus PITR.

## Preguntas de repaso

- Que diferencia fundamental hay entre un backup con `pg_dump` y PITR en cuanto a los puntos de restauracion posibles?
- Que rol cumple el WAL en hacer posible PITR?
- Para que tipo de incidente especifico es PITR la herramienta correcta que `pg_dump` no puede ofrecer?
- Por que es importante probar el proceso completo de PITR antes de necesitarlo en una situacion real de incidente?
