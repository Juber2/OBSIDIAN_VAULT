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

# 05 - Diseño para Aplicaciones NET

## Que vas a aprender

En este capitulo vas a cerrar el capitulo consolidando el diseño de esquemas especificamente pensado para aplicaciones .NET: columnas de auditoria, convenciones de naming (PascalCase de .NET vs snake_case de Postgres), y **Code First vs Database First** en Entity Framework Core.

## Explicacion

En proyectos .NET, la convencion de nombres tipica es **PascalCase** tanto en C# como en las tablas/columnas de SQL Server (`Empleados`, `FechaCreacion`), a diferencia de la convencion `snake_case` mas comun en el ecosistema Postgres/Node (`empleados`, `creado_en`, ya vista en ese libro). Esta diferencia de convencion no es tecnica sino cultural del ecosistema, y es importante mantenerla consistente dentro de cada proyecto.

Entity Framework Core soporta dos flujos de trabajo: **Code First** (se definen las clases C# primero, y EF Core genera migrations que crean el esquema SQL a partir de ellas, similar en espiritu a como TypeORM genera migrations desde entidades, ver libro de NestJS) y **Database First** (el esquema SQL ya existe, y se genera el modelo de clases C# a partir de el con scaffolding). Para proyectos nuevos como StaffTrainingPlatform, Code First es el flujo mas comun y recomendado, dando control total sobre el modelo desde el codigo versionado.

## Modelo mental

Pensa Code First como diseñar primero los planos arquitectonicos completos (las clases C#) y despues construir el edificio real (el esquema SQL) siguiendo esos planos exactos. Database First es el proceso inverso: el edificio ya existe (una base de datos legacy, por ejemplo), y se documentan sus planos despues de haberlo construido, util para integrarse con sistemas ya existentes.

## Ejemplo base

```csharp
// Code First: la clase C# define el esquema
public class Empleado
{
    public long Id { get; set; }
    public string Nombre { get; set; }
    public string Email { get; set; }
    public DateTime CreadoEn { get; set; } = DateTime.UtcNow;
    public DateTime? ActualizadoEn { get; set; }
}
```

```sql
-- Esquema generado automaticamente por una migration de EF Core (Code First)
CREATE TABLE Empleados (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    Nombre NVARCHAR(MAX) NOT NULL,
    Email NVARCHAR(MAX) NOT NULL,
    CreadoEn DATETIME2 NOT NULL,
    ActualizadoEn DATETIME2 NULL
);
```

## Como se usa en un proyecto real

1. Se usa Code First para proyectos .NET nuevos como StaffTrainingPlatform, definiendo el modelo en clases C# y generando migrations desde ahi.
2. Se mantiene PascalCase consistente en tablas y columnas SQL Server, alineado con la convencion de nombres de C#.
3. Se agregan columnas de auditoria estandar (`CreadoEn`, `ActualizadoEn`) a cada entidad, aplicando el mismo principio ya visto en Postgres.
4. Se documenta el diagrama de entidades completo del proyecto, sirviendo tanto como documentacion del esquema SQL como del modelo de clases C#.
5. Se evalua Database First solo al integrarse con bases de datos legacy ya existentes que no se controlan desde el codigo del proyecto.

## Criterio junior

Un junior debe poder definir una clase C# simple y generar el esquema T-SQL correspondiente con Code First.

## Criterio mid

Un mid debe mantener convenciones de naming consistentes (PascalCase) y agregar columnas de auditoria estandar a cada entidad.

## Criterio senior

Un senior debe decidir entre Code First y Database First segun el contexto del proyecto, y establecer las convenciones completas de diseño de esquema para el equipo .NET.

## Errores comunes

- Mezclar convenciones de nombres (PascalCase y snake_case) dentro del mismo proyecto SQL Server, generando inconsistencia.
- Usar Database First para un proyecto nuevo donde Code First daria mas control y mejor versionado del esquema desde el codigo.
- No agregar columnas de auditoria estandar desde el diseño inicial de cada entidad.
- No documentar el modelo de entidades completo, dificultando el entendimiento del esquema para nuevos integrantes.

## Practica

- [ ] Define una clase C# simple y genera su esquema T-SQL equivalente con Code First (conceptualmente, si no tenes el entorno completo).
- [ ] Verifica que las convenciones de naming son consistentes en un esquema real de tu proyecto.
- [ ] Agrega columnas de auditoria estandar a una entidad de practica.
- [ ] Compara Code First y Database First para el contexto de StaffTrainingPlatform.
- [ ] Documenta el diagrama de entidades de un dominio real de tu proyecto .NET.

## Preguntas de repaso

- Que diferencia de convencion de nombres es tipica entre proyectos .NET (SQL Server) y proyectos Node (Postgres)?
- Que diferencia hay entre el flujo Code First y Database First de Entity Framework Core?
- Cuando se justificaria usar Database First en vez de Code First?
- Que columnas de auditoria estandar agregarias a cada entidad de un proyecto .NET nuevo?
