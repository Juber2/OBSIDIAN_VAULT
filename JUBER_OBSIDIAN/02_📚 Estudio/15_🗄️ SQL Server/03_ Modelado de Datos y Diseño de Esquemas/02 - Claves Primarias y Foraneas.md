---
tags:
  - SQLServer
  - Modelado
  - SQLServer-Modelado
Creado: 2026-07-29
Relacionado:
  - Modelado de Datos y Diseño de Esquemas
  - SQL SERVER
---

# 02 - Claves Primarias y Foraneas

## Que vas a aprender

En este capitulo vas a profundizar en claves primarias (`IDENTITY` vs `UNIQUEIDENTIFIER`, el equivalente de UUID en SQL Server) y foraneas con `ON DELETE`/`ON UPDATE` en T-SQL, aplicando el mismo criterio de diseño ya establecido en Postgres.

## Explicacion

SQL Server ofrece `IDENTITY` (autoincremental, ya visto en el capitulo de Tipos de Datos) como equivalente directo del `bigint GENERATED ALWAYS AS IDENTITY` de Postgres, y **`UNIQUEIDENTIFIER`** (con `NEWID()` o `NEWSEQUENTIALID()` para generar valores) como equivalente del `uuid` de Postgres. Una particularidad de SQL Server: `NEWSEQUENTIALID()` genera GUIDs con un componente secuencial creciente, mitigando parcialmente el problema de fragmentacion de indices que un GUID completamente aleatorio causaria (ya mencionado como tradeoff de UUID en el libro de Postgres).

Las politicas `ON DELETE CASCADE`/`ON DELETE NO ACTION` (el equivalente de `RESTRICT` de Postgres) siguen exactamente la misma logica de diseño ya estudiada: `CASCADE` para relaciones de composicion real, `NO ACTION` (default) cuando la eliminacion accidental no deberia propagarse sin decision explicita.

## Modelo mental

El modelo mental es identico al ya establecido en Postgres: una clave foranea es la regla que impide referenciar algo que no existe, y las politicas de eliminacion definen que pasa con esas referencias cuando el original se elimina. La unica diferencia real es el vocabulario especifico (`UNIQUEIDENTIFIER` en vez de `uuid`, `NO ACTION` en vez de `RESTRICT`).

## Ejemplo base

```sql
CREATE TABLE Empleados (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    Nombre NVARCHAR(100) NOT NULL
);

CREATE TABLE Inscripciones (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    EmpleadoId BIGINT NOT NULL REFERENCES Empleados(Id) ON DELETE NO ACTION,
    CursoId BIGINT NOT NULL REFERENCES Cursos(Id) ON DELETE CASCADE
);

-- UNIQUEIDENTIFIER como alternativa de clave primaria
CREATE TABLE Sesiones (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWSEQUENTIALID(),
    EmpleadoId BIGINT REFERENCES Empleados(Id)
);
```

## Como se usa en un proyecto real

1. Se usa `BIGINT IDENTITY` como default para la mayoria de tablas en StaffTrainingPlatform, aplicando el mismo criterio ya establecido en Postgres.
2. Se elige `UNIQUEIDENTIFIER` con `NEWSEQUENTIALID()` (no `NEWID()` puro) cuando se necesita un GUID pero se quiere mitigar el problema de fragmentacion de indices.
3. Se define explicitamente la politica `ON DELETE` para cada relacion, aplicando el mismo criterio de "composicion real vs restriccion protectora" ya visto en Postgres.
4. Se conecta esto con `HasForeignKey()`/`OnDelete()` de Entity Framework Core (ver capitulo de SQL Server con .NET), donde estas mismas politicas se configuran desde C#.
5. Se aprovecha que el criterio de diseño es completamente transferible desde Postgres, solo cambiando el vocabulario T-SQL especifico.

## Criterio junior

Un junior debe poder definir una clave primaria con `IDENTITY` y una clave foranea basica con `REFERENCES` en T-SQL.

## Criterio mid

Un mid debe elegir entre `IDENTITY` y `UNIQUEIDENTIFIER` con criterio, y definir politicas `ON DELETE` explicitas para cada relacion.

## Criterio senior

Un senior debe diseñar la estrategia de claves primarias de un sistema .NET completo, y revisar en code review cualquier `CASCADE` no justificado.

## Errores comunes

- Usar `NEWID()` en vez de `NEWSEQUENTIALID()` sin considerar el impacto de fragmentacion de indices en tablas de alto volumen de insercion.
- Dejar el comportamiento por defecto (`NO ACTION`) sin decidir conscientemente la politica correcta para cada relacion.
- No transferir el criterio de diseño ya establecido en Postgres, tratando esta decision como si fuera nueva y sin precedente.
- No validar en pruebas el efecto real de un `CASCADE` antes de aplicarlo en produccion.

## Practica

- [ ] Define una relacion con `ON DELETE CASCADE` y otra con `ON DELETE NO ACTION` en T-SQL.
- [ ] Compara `NEWID()` y `NEWSEQUENTIALID()` para una clave primaria `UNIQUEIDENTIFIER`.
- [ ] Documenta el criterio para elegir entre `IDENTITY` y `UNIQUEIDENTIFIER` en un proyecto .NET real.
- [ ] Conecta la politica `ON DELETE` de una relacion con `OnDelete()` de Entity Framework Core.
- [ ] Revisa un esquema real de StaffTrainingPlatform (o similar) verificando politicas `ON DELETE` explicitas.

## Preguntas de repaso

- Que diferencia hay entre `NEWID()` y `NEWSEQUENTIALID()` para generar `UNIQUEIDENTIFIER`?
- Como se llama en T-SQL la politica equivalente a `RESTRICT` de Postgres?
- Que criterio usarias para elegir entre `IDENTITY` y `UNIQUEIDENTIFIER` como clave primaria?
- Como se configura la politica `ON DELETE` desde Entity Framework Core?
