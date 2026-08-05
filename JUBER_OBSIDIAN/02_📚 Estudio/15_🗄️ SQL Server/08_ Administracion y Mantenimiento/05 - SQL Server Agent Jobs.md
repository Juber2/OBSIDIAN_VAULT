---
tags:
  - SQLServer
  - Administracion
  - SQLServer-Administracion
Creado: 2026-07-29
Relacionado:
  - Administracion y Mantenimiento
  - SQL SERVER
---

# 05 - SQL Server Agent Jobs

## Que vas a cerrar

En este capitulo vas a cerrar el capitulo aprendiendo **SQL Server Agent**, el equivalente directo de cron (ya visto en profundidad en el libro de Linux) integrado nativamente dentro de SQL Server, para automatizar backups, mantenimiento, y cualquier tarea T-SQL programada.

## Explicacion

SQL Server Agent es un servicio que ejecuta **Jobs** programados segun un **Schedule** (horario o frecuencia definida), cada Job compuesto de uno o mas **Steps** (pasos, tipicamente comandos T-SQL, aunque tambien soporta scripts de PowerShell o ejecutables externos). Esto es funcionalmente identico en proposito a cron (ver libro de Linux), pero integrado directamente dentro del motor de base de datos, sin necesitar el sistema operativo del host para la programacion.

Los mismos principios ya vistos para cron aplican aca: los Jobs deben tener manejo de errores robusto (Agent puede notificar por email ante fallo, configurable), y las tareas criticas (backups, ver tema 1) deben monitorearse activamente para confirmar que efectivamente se ejecutaron con exito, no solo asumir que "deberian estar corriendo" sin verificarlo.

## Modelo mental

Pensa SQL Server Agent como un cron dedicado que vive directamente dentro de la base de datos, en vez de depender del sistema operativo del servidor: la misma idea de "despertador confiable que ejecuta tareas programadas" ya visto con cron en el libro de Linux, pero con acceso directo y nativo a T-SQL sin necesitar invocar herramientas externas.

## Ejemplo base

```sql
-- Crear un Job (via T-SQL, aunque tambien se configura visualmente en SSMS)
EXEC msdb.dbo.sp_add_job @job_name = 'BackupDiarioMiApp';

EXEC msdb.dbo.sp_add_jobstep
    @job_name = 'BackupDiarioMiApp',
    @step_name = 'EjecutarBackupFull',
    @command = 'BACKUP DATABASE MiApp TO DISK = ''C:\Backups\MiApp_Full.bak''';

EXEC msdb.dbo.sp_add_schedule
    @schedule_name = 'DiarioA3AM',
    @freq_type = 4,           -- diario
    @active_start_time = 030000;   -- 3:00 AM

EXEC msdb.dbo.sp_attach_schedule
    @job_name = 'BackupDiarioMiApp',
    @schedule_name = 'DiarioA3AM';
```

```sql
-- Verificar el historial de ejecucion de un Job
SELECT * FROM msdb.dbo.sysjobhistory
WHERE job_id = (SELECT job_id FROM msdb.dbo.sysjobs WHERE name = 'BackupDiarioMiApp');
```

## Como se usa en un proyecto real

1. Se automatiza toda la estrategia de backup (Full/Differential/Log, ver tema 1) como Agent Jobs con schedules apropiados, en vez de ejecucion manual.
2. Se configura notificacion por email ante fallo de un Job critico, para que un humano se entere activamente sin necesitar revisar manualmente el historial.
3. Se revisa `sysjobhistory` regularmente para verificar que los Jobs criticos efectivamente se ejecutaron con exito, aplicando el mismo principio de verificacion activa ya insistido en el libro de Linux.
4. Se conecta esto directamente con cron del libro de Linux, entendiendo que resuelven el mismo problema de automatizacion con mecanismos analogos.
5. Se documenta cada Job de mantenimiento critico (proposito, horario, que hacer si falla) para el equipo de infraestructura.

## Criterio junior

Un junior debe poder crear un Agent Job simple con un schedule basico.

## Criterio mid

Un mid debe configurar notificaciones ante fallo de Jobs criticos, y revisar el historial de ejecucion regularmente.

## Criterio senior

Un senior debe diseñar la suite completa de automatizacion de mantenimiento (backups, indices, estadisticas) como Agent Jobs coordinados, con monitoreo activo de su ejecucion exitosa.

## Errores comunes

- No configurar notificaciones ante fallo de un Job critico, descubriendo que un backup dejo de ejecutarse solo cuando ya se necesita restaurar.
- No revisar el historial de ejecucion regularmente, asumiendo que los Jobs "deberian estar funcionando" sin verificarlo activamente.
- Configurar Jobs sin manejo de errores robusto en cada Step, dejando que fallen silenciosamente sin informacion util de diagnostico.
- No documentar Jobs criticos, dejando que solo una persona entienda el proposito y comportamiento esperado de cada uno.

## Practica

- [ ] Crea un Agent Job simple con un Step de T-SQL y un Schedule diario.
- [ ] Configura notificacion por email ante fallo (o investiga como lo harias).
- [ ] Revisa `sysjobhistory` para verificar el historial de ejecucion.
- [ ] Diseña la suite completa de Jobs de mantenimiento (backup, indices, estadisticas) para un sistema de produccion.
- [ ] Compara SQL Server Agent con cron, identificando las equivalencias directas.

## Preguntas de repaso

- Que es un Job, un Step, y un Schedule en SQL Server Agent?
- Por que es importante configurar notificaciones ante fallo de un Job critico?
- Como se relaciona SQL Server Agent con cron, ya estudiado en el libro de Linux?
- Que informacion revisarias en `sysjobhistory` para verificar la salud de un Job?

## Cierre del capitulo

Con este tema cierras el capitulo de administracion de SQL Server, habiendo aplicado el mismo rigor de backup, seguridad, y mantenimiento ya establecido en Postgres, con las herramientas y particularidades especificas de SQL Server, incluyendo su integracion nativa de automatizacion con SQL Server Agent.
