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

# 01 - Backup Full Differential Log

## Que vas a aprender

En este capitulo vas a aprender el modelo de backup de SQL Server, mas granular que el `pg_dump` simple de Postgres ya estudiado: **Full**, **Differential**, y **Transaction Log** backups, combinados en una estrategia real de produccion.

## Explicacion

Un backup **Full** respalda la base de datos completa en un momento dado (equivalente conceptual a `pg_dump`). Un backup **Differential** respalda solo los cambios desde el **ultimo Full** (no desde el ultimo differential), siendo mas rapido de generar que un Full completo pero acumulando mas datos con cada dia que pasa desde el ultimo Full. Un backup de **Transaction Log** respalda los cambios registrados en el log desde el ultimo backup de log (o desde el ultimo Full/Differential), permitiendo restaurar a un punto exacto en el tiempo, el equivalente directo del PITR con WAL ya estudiado en profundidad en el libro de Postgres.

Una estrategia tipica de produccion combina los tres: Full semanal, Differential diario, y Transaction Log cada 15-30 minutos, permitiendo restaurar con precision minimizando tanto el tiempo de backup diario como la cantidad de datos potencialmente perdidos ante un incidente.

## Modelo mental

Pensa un backup Full como una fotografia completa del archivo entero. Un backup Differential es como fotografiar solo las paginas que cambiaron desde la ultima fotografia completa (crece cada dia hasta el proximo Full). Un backup de Transaction Log es como una filmacion continua de cada cambio individual, permitiendo "rebobinar" hasta un segundo exacto, exactamente el mismo concepto ya dominado con PITR de Postgres, con nombres distintos.

## Ejemplo base

```sql
-- Backup Full
BACKUP DATABASE MiApp TO DISK = 'C:\Backups\MiApp_Full.bak';

-- Backup Differential (requiere un Full previo)
BACKUP DATABASE MiApp TO DISK = 'C:\Backups\MiApp_Diff.bak' WITH DIFFERENTIAL;

-- Backup de Transaction Log (requiere recovery model FULL, ver siguiente tema)
BACKUP LOG MiApp TO DISK = 'C:\Backups\MiApp_Log.trn';
```

## Como se usa en un proyecto real

1. Se diseña una estrategia combinada (Full semanal, Differential diario, Log cada 15-30 minutos) para bases de datos criticas de produccion, equilibrando velocidad de backup con precision de recuperacion.
2. Se automatiza esta estrategia con SQL Server Agent (ver tema de Agent Jobs) en vez de ejecucion manual.
3. Se conecta esto directamente con PITR de Postgres, entendiendo que ambos motores resuelven el mismo problema (recuperacion a un punto exacto en el tiempo) con mecanismos analogos.
4. Se prueba regularmente la restauracion completa de esta cadena (Full + Differential + Logs), aplicando el mismo principio ya insistido en el libro de Postgres.
5. Se documenta el RPO/RTO real que esta estrategia garantiza, comunicandolo al negocio.

## Criterio junior

Un junior debe poder generar un backup Full basico y entender la diferencia conceptual con Differential y Log.

## Criterio mid

Un mid debe diseñar una estrategia combinada de Full/Differential/Log para un sistema de produccion real.

## Criterio senior

Un senior debe garantizar pruebas regulares de restauracion completa de esta cadena, y definir RPO/RTO documentados para sistemas criticos.

## Errores comunes

- No entender que un Differential respalda desde el ultimo Full (no desde el ultimo Differential), calculando mal la cadena de restauracion necesaria.
- No respaldar el Transaction Log regularmente, perdiendo la capacidad de recuperacion a un punto exacto en el tiempo.
- No probar la restauracion completa de la cadena Full+Differential+Log, descubriendo problemas solo durante un incidente real.
- Guardar todos los backups en el mismo disco que la base de datos original, violando la regla 3-2-1 ya vista en el libro de Linux.

## Practica

- [ ] Genera un backup Full de una base de datos de practica.
- [ ] Genera un backup Differential despues de algunos cambios.
- [ ] Configura y genera un backup de Transaction Log.
- [ ] Diseña una estrategia completa combinada para un sistema de produccion hipotetico.
- [ ] Compara esta estrategia con PITR de Postgres, identificando las equivalencias.

## Preguntas de repaso

- Desde que punto de referencia respalda un backup Differential?
- Que permite recuperar un backup de Transaction Log que un Full/Differential no puede por si solo?
- Como se relaciona esta estrategia de backup con PITR de Postgres?
- Que estrategia combinada (frecuencia de cada tipo) diseñarias para un sistema critico de produccion?
