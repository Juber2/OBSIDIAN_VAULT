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

# 02 - Restore y Recovery Models

## Que vas a aprender

En este capitulo vas a aprender los **Recovery Models** de SQL Server (`SIMPLE`, `FULL`, `BULK_LOGGED`), que determinan que capacidades de backup/restore estan disponibles, y el proceso de `RESTORE` completo usando la cadena de backups del tema anterior.

## Explicacion

El **Recovery Model** de una base de datos determina como se gestiona el Transaction Log: **SIMPLE** trunca el log automaticamente despues de cada checkpoint, **sin permitir** backups de Transaction Log (por lo tanto, sin PITR real, solo recuperable hasta el ultimo Full/Differential), apropiado para bases de datos de desarrollo o donde la perdida de datos recientes es aceptable. **FULL** mantiene el log completo hasta que se respalda explicitamente, **habilitando** backups de log y por lo tanto PITR real, el modelo correcto para cualquier base de datos de produccion critica.

El proceso de `RESTORE` reconstruye la base de datos aplicando la cadena completa: `RESTORE DATABASE ... WITH NORECOVERY` (deja la base en estado de "esperando mas backups") para el Full, luego el Differential mas reciente, y finalmente los Transaction Logs en orden hasta el punto deseado, terminando con `WITH RECOVERY` (o `STOPAT` para PITR) en el ultimo paso.

## Modelo mental

Pensa el Recovery Model como decidir si guardas o descartas el borrador completo de un documento despues de cada version guardada: SIMPLE tira el borrador (no podes reconstruir cambios intermedios especificos), FULL guarda cada borrador intermedio (permitiendo reconstruir exactamente el estado en cualquier momento intermedio, como PITR de Postgres).

## Ejemplo base

```sql
-- Configurar Recovery Model
ALTER DATABASE MiApp SET RECOVERY FULL;

-- Restaurar la cadena completa: Full -> Differential -> Logs
RESTORE DATABASE MiApp FROM DISK = 'C:\Backups\MiApp_Full.bak' WITH NORECOVERY;
RESTORE DATABASE MiApp FROM DISK = 'C:\Backups\MiApp_Diff.bak' WITH NORECOVERY;
RESTORE LOG MiApp FROM DISK = 'C:\Backups\MiApp_Log1.trn' WITH NORECOVERY;
RESTORE LOG MiApp FROM DISK = 'C:\Backups\MiApp_Log2.trn'
    WITH RECOVERY, STOPAT = '2026-07-29 14:59:00';   -- PITR: hasta justo antes de un error
```

## Como se usa en un proyecto real

1. Se configura `FULL` recovery model para toda base de datos de produccion critica, incluyendo StaffTrainingPlatform, habilitando la capacidad de PITR real.
2. Se usa `SIMPLE` solo para bases de datos de desarrollo o donde la perdida de datos desde el ultimo Full/Differential es genuinamente aceptable.
3. Se practica el proceso completo de `RESTORE` con `NORECOVERY`/`WITH RECOVERY` regularmente, entendiendo que un error en la secuencia deja la base en estado inconsistente.
4. Se conecta esto directamente con PITR de Postgres, usando `STOPAT` de forma identica en proposito a `recovery_target_time` ya visto en ese libro.
5. Se documenta el procedimiento exacto de restauracion paso a paso para cada sistema critico, dado que es una operacion de alto estres ejecutada raramente.

## Criterio junior

Un junior debe poder configurar un Recovery Model y entender la diferencia entre SIMPLE y FULL.

## Criterio mid

Un mid debe ejecutar una restauracion completa de la cadena Full+Differential+Log con `NORECOVERY`/`WITH RECOVERY`.

## Criterio senior

Un senior debe diseñar y probar regularmente el procedimiento completo de PITR para sistemas criticos, documentandolo para uso en situaciones de alta presion.

## Errores comunes

- Dejar una base de datos de produccion critica en `SIMPLE` recovery model, perdiendo la capacidad de PITR real ante un error descubierto tarde.
- No entender el orden correcto de `NORECOVERY`/`WITH RECOVERY` en una restauracion multi-paso, dejando la base en un estado inconsistente.
- No probar nunca el proceso completo de restauracion con PITR, descubriendo problemas de configuracion recien durante un incidente real.
- No documentar el procedimiento exacto de restauracion, dependiendo de que una unica persona recuerde el proceso bajo presion.

## Practica

- [ ] Configura `FULL` recovery model en una base de datos de practica.
- [ ] Ejecuta una restauracion completa de la cadena Full+Differential+Log.
- [ ] Practica una restauracion con PITR usando `STOPAT` a un momento especifico.
- [ ] Documenta el procedimiento completo paso a paso que seguiste.
- [ ] Compara este proceso con PITR de Postgres, identificando las equivalencias exactas.

## Preguntas de repaso

- Que diferencia hay entre Recovery Model SIMPLE y FULL en cuanto a capacidades de backup/restore?
- Por que toda base de datos de produccion critica deberia usar FULL recovery model?
- Que hace `NORECOVERY` durante una restauracion multi-paso, y por que es necesario?
- Como se relaciona `STOPAT` de SQL Server con `recovery_target_time` de Postgres?
