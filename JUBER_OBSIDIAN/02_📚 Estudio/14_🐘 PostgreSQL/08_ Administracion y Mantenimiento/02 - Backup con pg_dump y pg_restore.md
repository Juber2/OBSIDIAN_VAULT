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

# 02 - Backup con pg_dump y pg_restore

## Que vas a aprender

En este capitulo vas a dominar `pg_dump`/`pg_restore`, las herramientas oficiales de backup logico de Postgres, conectando con el patron general de backup ya visto en el libro de Linux, ahora especifico del motor.

## Explicacion

`pg_dump` genera un **backup logico**: un archivo con las sentencias SQL (o un formato binario propio mas eficiente) necesarias para reconstruir la base de datos completa, distinto de un backup fisico (copiar directamente los archivos de datos del disco). El formato **plano** (`-Fp`, SQL legible) es simple pero mas lento de restaurar en bases grandes; el formato **custom** (`-Fc`) es comprimido y permite restauracion **selectiva** (solo ciertas tablas) y **paralela** (`pg_restore -j N`), mucho mas rapida en bases de datos grandes.

`pg_dumpall` respalda el cluster completo incluyendo roles y configuracion global, mientras que `pg_dump` respalda una unica base de datos especifica. Para bases de datos muy grandes donde `pg_dump` se vuelve lento, la alternativa es backup fisico basado en el **WAL** (ver siguiente tema, Point-in-Time Recovery), que ademas permite restaurar a un momento exacto en el tiempo, no solo al momento en que se genero el backup.

## Modelo mental

Pensa `pg_dump` como fotografiar cada pagina de un libro y anotar las instrucciones exactas para reconstruirlo desde cero (backup logico), mientras que un backup fisico es directamente fotocopiar el libro completo tal cual esta impreso. El formato custom con restauracion paralela es como tener ese libro fotografiado con un indice detallado que permite reconstruir solo el capitulo que necesitas, en vez de tener que reconstruir el libro entero cada vez.

## Ejemplo base

```bash
pg_dump -Fc -f backup.dump mi_basededatos     # formato custom comprimido
pg_dump -Fp -f backup.sql mi_basededatos       # formato plano SQL legible

pg_dumpall -f backup_completo.sql               # cluster completo, roles incluidos

pg_restore -d mi_basededatos_nueva backup.dump          # restauracion completa
pg_restore -d mi_basededatos_nueva -j 4 backup.dump       # restauracion paralela, 4 jobs
pg_restore -d mi_basededatos_nueva -t pedidos backup.dump  # restaura solo una tabla especifica
```

## Como se usa en un proyecto real

1. Se usa formato custom (`-Fc`) como default para backups de produccion, aprovechando compresion y restauracion selectiva/paralela sobre el formato plano.
2. Se automatiza `pg_dump` como tarea de cron (ver capitulo de Scripting del libro de Linux), con logging robusto y rotacion de backups viejos.
3. Se prueba la restauracion regularmente en un ambiente de prueba, aplicando el mismo principio de "probar el backup, no solo generarlo" visto en el libro de Linux.
4. Se evalua backup fisico basado en WAL (ver tema siguiente) cuando `pg_dump` se vuelve demasiado lento para el volumen de datos, o cuando se necesita capacidad de recuperacion a un punto exacto en el tiempo.
5. Se envia el backup a almacenamiento externo (siguiendo la regla 3-2-1 ya vista en el libro de Linux), nunca solo al mismo servidor que aloja la base de datos original.

## Criterio junior

Un junior debe poder generar un backup con `pg_dump` y restaurarlo con `pg_restore` en una base de datos vacia.

## Criterio mid

Un mid debe automatizar backups con `pg_dump` en formato custom, y usar restauracion selectiva/paralela para bases de datos grandes.

## Criterio senior

Un senior debe diseñar la estrategia completa de backup de Postgres del sistema, decidiendo entre backup logico y fisico segun el volumen de datos y los requisitos de RPO/RTO, y garantizando pruebas regulares de restauracion.

## Errores comunes

- Usar formato plano para bases de datos grandes, sufriendo tiempos de restauracion mucho mayores de lo necesario frente al formato custom.
- No probar la restauracion regularmente, descubriendo un backup corrupto o incompleto solo durante un incidente real.
- Guardar backups solo en el mismo servidor que la base de datos original, perdiendo ambos ante una falla completa de ese servidor.
- No automatizar el proceso de backup, dependiendo de ejecucion manual que puede olvidarse.

## Practica

- [ ] Genera un backup de una base de datos de practica con `pg_dump -Fc`.
- [ ] Restaura ese backup en una base de datos nueva con `pg_restore`.
- [ ] Compara el tiempo de restauracion con `-j` (paralelo) versus sin esa opcion.
- [ ] Restaura selectivamente solo una tabla especifica del backup completo.
- [ ] Automatiza el proceso de backup como una tarea de cron con logging.

## Preguntas de repaso

- Que ventaja da el formato custom (`-Fc`) sobre el formato plano (`-Fp`) de `pg_dump`?
- Que diferencia hay entre `pg_dump` (una base de datos) y `pg_dumpall` (cluster completo)?
- Como aprovecharias restauracion paralela para acelerar la recuperacion de una base de datos grande?
- Por que probar la restauracion regularmente es tan importante como generar el backup en si?
