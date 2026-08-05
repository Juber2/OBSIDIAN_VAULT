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

# 04 - Relaciones 1a1 1aN NaM

## Que vas a aprender

En este capitulo vas a modelar relaciones uno a uno, uno a muchos y muchos a muchos en T-SQL, aplicando exactamente la misma teoria relacional ya dominada en Postgres, ahora conectada directamente con Entity Framework Core.

## Explicacion

Los tres tipos de relacion se modelan identicamente en T-SQL: uno a muchos con clave foranea en el lado "muchos", uno a uno agregando `UNIQUE` a esa clave foranea, y muchos a muchos con una tabla intermedia. La unica diferencia real frente a Postgres es sintactica (tipos de datos, `IDENTITY` en vez de `GENERATED ALWAYS AS IDENTITY`).

Lo especifico de este capitulo para tu contexto es la conexion directa con **Entity Framework Core**: una relacion uno a muchos en T-SQL corresponde a una **navigation property** de coleccion en la entidad C# (`ICollection<Inscripcion>`), una relacion uno a uno a una navigation property simple opcional, y una relacion muchos a muchos puede modelarse explicitamente con una entidad intermedia (recomendado cuando esa relacion tiene datos propios) o de forma implicita que EF Core gestiona automaticamente (para casos simples sin datos adicionales en la relacion).

## Modelo mental

El modelo mental es identico al ya establecido en Postgres. Lo nuevo aca es la traduccion directa hacia C#: pensa cada tabla como una clase, cada relacion uno a muchos como una lista dentro de esa clase apuntando a otras instancias, y la tabla intermedia de una relacion muchos a muchos como una clase propia con sus dos referencias, cuando esa relacion necesita tener datos adicionales.

## Ejemplo base

```sql
-- Uno a muchos: un empleado tiene muchas inscripciones
CREATE TABLE Inscripciones (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    EmpleadoId BIGINT NOT NULL REFERENCES Empleados(Id)
);

-- Muchos a muchos con datos propios: cursos y empleados via inscripciones (con fecha y calificacion)
CREATE TABLE InscripcionesCursos (
    EmpleadoId BIGINT NOT NULL REFERENCES Empleados(Id),
    CursoId BIGINT NOT NULL REFERENCES Cursos(Id),
    FechaInscripcion DATE NOT NULL DEFAULT GETDATE(),
    Calificacion DECIMAL(4,2),
    PRIMARY KEY (EmpleadoId, CursoId)
);
```

```csharp
// Entity Framework Core: navigation properties correspondientes
public class Empleado
{
    public long Id { get; set; }
    public ICollection<Inscripcion> Inscripciones { get; set; }
}
```

## Como se usa en un proyecto real

1. Se modela cada relacion uno a muchos con la clave foranea en el lado correcto, aplicando el mismo criterio ya establecido en Postgres.
2. Se usa una tabla intermedia explicita con clase de entidad propia en EF Core cuando la relacion muchos a muchos tiene datos propios (fecha, calificacion), como en el caso de inscripciones de StaffTrainingPlatform.
3. Se conecta cada relacion T-SQL con la navigation property C# correspondiente, entendiendo el mapeo completo entre SQL y el modelo de objetos.
4. Se documenta el diagrama entidad-relacion del esquema completo del proyecto.
5. Se aprovecha que este conocimiento es directamente transferible desde Postgres, acelerando significativamente el aprendizaje de este tema especifico.

## Criterio junior

Un junior debe poder modelar una relacion uno a muchos basica en T-SQL y su navigation property equivalente en C#.

## Criterio mid

Un mid debe modelar relaciones muchos a muchos con tabla intermedia explicita cuando la relacion tiene datos propios.

## Criterio senior

Un senior debe diseñar el modelo relacional completo de un sistema .NET complejo, revisando la correspondencia correcta entre esquema SQL y entidades EF Core.

## Errores comunes

- Modelar una relacion muchos a muchos sin tabla intermedia explicita cuando en realidad necesita datos propios (fecha, estado), perdiendo esa informacion.
- No conectar correctamente el esquema SQL con las navigation properties de EF Core, generando mapeo incorrecto o warnings de configuracion.
- Poner la clave foranea en el lado equivocado de una relacion uno a muchos.
- No aprovechar el conocimiento ya solido de relaciones de Postgres, tratando este tema como completamente nuevo.

## Practica

- [ ] Modela una relacion uno a muchos en T-SQL y su navigation property equivalente en C#.
- [ ] Modela una relacion muchos a muchos con tabla intermedia explicita que tenga datos propios.
- [ ] Compara la sintaxis de relaciones de T-SQL con la ya estudiada en Postgres.
- [ ] Diseña el modelo de relaciones completo de un dominio real de StaffTrainingPlatform.
- [ ] Conecta cada relacion con su equivalente exacto en Entity Framework Core.

## Preguntas de repaso

- Por que la teoria de relaciones (1a1, 1aN, NaM) es identica entre T-SQL y Postgres?
- Cuando conviene una tabla intermedia explicita para una relacion muchos a muchos en vez de la gestion implicita de EF Core?
- Como se traduce una relacion uno a muchos de SQL a una navigation property de C#?
- Que informacion adicional justificaria modelar una relacion muchos a muchos con datos propios?
