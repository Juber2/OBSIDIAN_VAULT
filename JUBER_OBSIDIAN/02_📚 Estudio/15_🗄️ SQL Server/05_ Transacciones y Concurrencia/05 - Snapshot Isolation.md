---
tags:
  - SQLServer
  - Transacciones
  - SQLServer-Transacciones
Creado: 2026-07-29
Relacionado:
  - Transacciones y Concurrencia
  - SQL SERVER
---

# 05 - Snapshot Isolation

## Que vas a aprender

En este capitulo vas a cerrar el capitulo profundizando en **Snapshot Isolation** y **Read Committed Snapshot Isolation (RCSI)**, las dos formas en que SQL Server implementa un modelo de versionado similar a MVCC de Postgres (ya visto en profundidad en ese libro), acercando el comportamiento de SQL Server al que ya conoces.

## Explicacion

Ya se menciono Snapshot Isolation en el tema de niveles de aislamiento; aca se profundiza en su mecanismo real: cuando se activa, SQL Server mantiene **versiones anteriores de filas modificadas** en una estructura llamada **tempdb version store** (conceptualmente similar a como Postgres mantiene versiones multiples de una fila via MVCC, ver ese libro), permitiendo que lectores vean una "fotografia" consistente sin bloquear a escritores concurrentes.

**RCSI** (`READ_COMMITTED_SNAPSHOT`) es una variante mas sutil: cambia el comportamiento del nivel Read Committed **por defecto** (sin que cada transaccion necesite pedir explicitamente el nivel Snapshot) para que las lecturas usen versionado en vez de locking tradicional, siendo la forma mas simple de acercar el comportamiento general de una base de datos completa al de Postgres, sin requerir cambios en cada consulta individual de la aplicacion.

## Modelo mental

Pensa RCSI como cambiar la configuracion por defecto de todo el restaurante (la base de datos) para que, en vez de que un mesero (lector) tenga que esperar a que la cocina (escritor) termine de preparar un plato antes de poder siquiera mirar la mesa, cada mesero automaticamente reciba una fotografia instantanea del estado de la mesa en el momento en que llego, sin bloquear a la cocina en absoluto: es el comportamiento de MVCC de Postgres, activado como default de toda la base de datos.

## Ejemplo base

```sql
-- Activar RCSI: cambia el comportamiento DEFAULT de Read Committed para toda la base de datos
ALTER DATABASE MiApp SET READ_COMMITTED_SNAPSHOT ON;

-- Ahora, sin necesitar SET TRANSACTION ISOLATION LEVEL explicito,
-- las lecturas usan versionado en vez de locks tradicionales
SELECT * FROM Pedidos WHERE Id = 1;   -- no bloquea ni es bloqueado

-- Monitorear el uso de tempdb version store (relevante porque RCSI/Snapshot lo usan activamente)
SELECT * FROM sys.dm_tran_version_store_space_usage;
```

## Como se usa en un proyecto real

1. Se evalua activar RCSI en bases de datos de produccion .NET (como StaffTrainingPlatform) donde la contencion de locking tradicional es un problema medido, acercando el comportamiento al ya familiar de Postgres.
2. Se monitorea el uso de `tempdb` (donde vive el version store) despues de activar RCSI/Snapshot, dado que mantener versiones adicionales consume espacio y recursos de esa base de sistema.
3. Se prueba el impacto en un ambiente de staging antes de activar en produccion, dado que cambia el comportamiento de concurrencia de toda la base de datos.
4. Se conecta esto directamente con el capitulo de MVCC de Postgres, entendiendo que RCSI es la forma en que SQL Server puede aproximarse voluntariamente a ese mismo modelo.
5. Se documenta la decision de activar (o no) RCSI como una decision arquitectonica real, con el mismo rigor ya aplicado en toda esta serie de libros.

## Criterio junior

Un junior debe entender conceptualmente que RCSI acerca el comportamiento de SQL Server al modelo MVCC ya conocido de Postgres.

## Criterio mid

Un mid debe activar RCSI en un ambiente de prueba y verificar el cambio de comportamiento de concurrencia resultante.

## Criterio senior

Un senior debe decidir si activar RCSI para una base de datos de produccion completa, monitoreando el impacto en `tempdb` y validando en staging antes del cambio.

## Errores comunes

- No conocer RCSI como opcion, sufriendo contencion de locking tradicional que Postgres nunca tendria para el mismo patron de uso.
- Activar RCSI en produccion sin haber probado el impacto en `tempdb` primero en un ambiente de staging.
- Confundir Snapshot Isolation (requiere `SET` explicito por transaccion) con RCSI (cambia el default de toda la base de datos automaticamente).
- No monitorear el version store despues de activar RCSI, perdiendo visibilidad de su impacto real en recursos.

## Practica

- [ ] Activa RCSI en una base de datos de practica y compara el comportamiento de concurrencia antes/despues.
- [ ] Monitorea `tempdb` version store durante una carga de trabajo concurrente con RCSI activo.
- [ ] Compara explicitamente RCSI con Snapshot Isolation explicito, entendiendo la diferencia.
- [ ] Conecta este capitulo completo con el modelo MVCC ya dominado en Postgres.
- [ ] Documenta si activarias RCSI para StaffTrainingPlatform y por que.

## Preguntas de repaso

- Que diferencia hay entre Snapshot Isolation explicito y RCSI?
- Como se relaciona el tempdb version store con el mecanismo de versiones multiples de MVCC en Postgres?
- Que verificarias antes de activar RCSI en una base de datos de produccion?
- Por que RCSI es la forma mas simple de acercar el comportamiento de SQL Server al de Postgres sin cambiar cada consulta individual?
