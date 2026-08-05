---
tags:
  - SQLServer
  - SeniorSQLServer
  - SQLServer-Senior
Creado: 2026-07-29
Relacionado:
  - Senior SQL Server
  - SQL SERVER
---

# 03 - Migraciones Zero-Downtime

## Que vas a aprender

En este capitulo vas a aprender a diseñar migraciones de esquema en SQL Server sin interrumpir la disponibilidad del sistema, aplicando directamente los mismos principios ya dominados en profundidad en el capitulo Senior de Postgres, con las particularidades del modelo de locking de SQL Server.

## Explicacion

El principio central es identico al ya estudiado en Postgres: cambios de esquema aditivos y en pasos pequeños son seguros, cambios destructivos o que bloquean la tabla completa durante mucho tiempo no lo son. En SQL Server especificamente, agregar una columna `NOT NULL` con un valor `DEFAULT` es una operacion **online y casi instantanea** desde SQL Server 2012 en adelante (el motor no necesita reescribir cada fila existente inmediatamente, similar a la optimizacion ya vista en Postgres 11+), pero agregar una columna `NOT NULL` **sin** default, o modificar el tipo de una columna existente, si requiere reescribir la tabla completa y puede tomar un lock prolongado sobre tablas grandes.

Para renombrar o eliminar una columna en uso, se aplica el mismo patron de migracion en varios pasos ya estudiado en Postgres: (1) agregar la columna nueva, (2) desplegar codigo que escribe en ambas columnas (vieja y nueva) simultaneamente, (3) hacer backfill de datos historicos, (4) desplegar codigo que lee solo de la columna nueva, (5) eliminar la columna vieja en una migracion separada posterior, nunca todo en un unico paso. Para crear indices sin bloquear escrituras en tablas grandes, se usa `CREATE INDEX ... WITH (ONLINE = ON)` (disponible en Enterprise Edition, y en Standard Edition desde SQL Server 2019), el equivalente directo a `CREATE INDEX CONCURRENTLY` de Postgres ya estudiado.

## Modelo mental

Pensa una migracion zero-downtime como remodelar una tienda que nunca puede cerrar sus puertas: no derribas una pared de un golpe (cambio destructivo instantaneo) mientras hay clientes adentro, la reorganizas en pasos pequeños y reversibles, siempre dejando un camino funcional para los clientes actuales mientras el cambio se completa gradualmente. Este es exactamente el mismo modelo mental ya aplicado en el libro de Postgres, con la sintaxis especifica de SQL Server para cada paso.

## Ejemplo base

```sql
-- Seguro: agregar columna NOT NULL con DEFAULT (online, rapido incluso en tablas grandes)
ALTER TABLE Cursos ADD Activo BIT NOT NULL DEFAULT 1;

-- Riesgoso en tablas grandes: crear indice bloqueando escrituras
CREATE INDEX IX_Cursos_Categoria ON Cursos(CategoriaId);

-- Seguro: crear indice sin bloquear escrituras (Enterprise, o Standard desde 2019)
CREATE INDEX IX_Cursos_Categoria ON Cursos(CategoriaId) WITH (ONLINE = ON);

-- Patron multi-paso para renombrar una columna en uso:
-- Paso 1: agregar columna nueva
ALTER TABLE Cursos ADD NombreCompleto NVARCHAR(200) NULL;
-- Paso 2 (deploy de app): escribir en ambas columnas
-- Paso 3: backfill de datos historicos
UPDATE Cursos SET NombreCompleto = Nombre WHERE NombreCompleto IS NULL;
-- Paso 4 (deploy de app): leer solo de la columna nueva
-- Paso 5 (migracion separada, despues de confirmar que nada usa la columna vieja):
ALTER TABLE Cursos DROP COLUMN Nombre;
```

## Como se usa en un proyecto real

1. Se diseñan todas las migraciones de StaffTrainingPlatform (via EF Core Migrations, ver capitulo de .NET) evaluando primero si el cambio es aditivo y seguro, o si requiere el patron multi-paso.
2. Se usa `WITH (ONLINE = ON)` al crear indices sobre tablas con trafico activo de escritura, evitando bloquear la aplicacion en produccion durante el proceso.
3. Se aplica el patron de renombrado en varios pasos (agregar, sincronizar, backfill, migrar lectura, eliminar) para cualquier cambio que afecte una columna en uso activo, nunca en un unico paso destructivo.
4. Se conecta esto directamente con el mismo principio ya dominado en el capitulo Senior de Postgres, reconociendo que la disciplina es identica aunque la sintaxis de cada paso cambie.
5. Se revisa cada script de migracion generado (ver capitulo de migraciones EF Core en la nube) especificamente buscando cambios potencialmente bloqueantes antes de aplicarlos contra produccion.

## Criterio junior

Un junior debe reconocer que agregar una columna `NOT NULL` con `DEFAULT` es seguro, mientras que modificar tipos o eliminar columnas en uso requiere mas cuidado.

## Criterio mid

Un mid debe aplicar el patron multi-paso para renombrar o eliminar columnas en uso activo, y usar `WITH (ONLINE = ON)` para creacion de indices en tablas grandes.

## Criterio senior

Un senior debe revisar y aprobar cada migracion de esquema en produccion evaluando su impacto de locking, diseñando el patron multi-paso apropiado para cambios de alto riesgo.

## Errores comunes

- Ejecutar `CREATE INDEX` sin `WITH (ONLINE = ON)` sobre una tabla grande en produccion, bloqueando escrituras durante un tiempo significativo.
- Eliminar o renombrar una columna en un unico paso mientras el codigo de la aplicacion aun la referencia, causando errores inmediatos en produccion.
- No verificar si `ONLINE = ON` esta disponible en la edicion de SQL Server usada (limitado en Standard antes de 2019).
- Asumir que todos los `ALTER TABLE` son igual de seguros, sin distinguir entre operaciones que reescriben la tabla completa y las que son metadatos-only.

## Practica

- [ ] Diseña una migracion segura que agregue una columna `NOT NULL` con `DEFAULT` a una tabla con datos existentes.
- [ ] Crea un indice con `WITH (ONLINE = ON)` y compara el comportamiento de locking con la version sin esa opcion.
- [ ] Diseña el patron multi-paso completo para renombrar una columna en uso activo.
- [ ] Revisa una Migration de EF Core generada automaticamente, identificando si contiene cambios potencialmente bloqueantes.
- [ ] Compara este proceso con migraciones zero-downtime ya dominadas en el capitulo Senior de Postgres.

## Preguntas de repaso

- Por que agregar una columna `NOT NULL` con `DEFAULT` es seguro desde SQL Server 2012 en adelante?
- Cuales son los cinco pasos del patron multi-paso para renombrar una columna en uso?
- Que ventaja ofrece `WITH (ONLINE = ON)` al crear un indice, y en que ediciones esta disponible?
- Como se compara esta disciplina con las migraciones zero-downtime ya estudiadas en Postgres?
