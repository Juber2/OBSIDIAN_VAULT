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

# 01 - Normalizacion

## Que vas a aprender

En este capitulo vas a aplicar las **formas normales** (1FN, 2FN, 3FN) ya estudiadas en profundidad en el libro de Postgres, ahora al diseño de esquemas T-SQL para proyectos .NET como StaffTrainingPlatform.

## Explicacion

La normalizacion es un concepto de **teoria relacional universal**, identico sin importar el motor: minimizar redundancia y evitar anomalias de actualizacion organizando datos en 1FN (valores atomicos), 2FN (dependencia completa de la clave), y 3FN (sin dependencias transitivas), exactamente como ya se estudio en profundidad en el libro de PostgreSQL. Lo que cambia entre motores es solo la sintaxis de implementacion (T-SQL en vez de SQL de Postgres), no el principio de diseño en si.

Para un proyecto real como StaffTrainingPlatform (.NET + SQL Server), el proceso de diseño es identico: identificar entidades del dominio (capacitaciones, empleados, cursos), sus atributos, y sus relaciones, aplicando despues normalizacion antes de traducir a `CREATE TABLE` en T-SQL.

## Modelo mental

El modelo mental es identico al ya establecido en el libro de Postgres: normalizar es asegurar que cada dato importante viva en un unico lugar autorizado, evitando que una actualizacion necesite corregir multiples copias dispersas del mismo dato.

## Ejemplo base

```sql
-- Mal (no normalizado): datos del empleado repetidos en cada inscripcion
CREATE TABLE InscripcionesMal (
    Id INT IDENTITY PRIMARY KEY,
    EmpleadoNombre NVARCHAR(100),
    EmpleadoEmail NVARCHAR(255),
    CursoNombre NVARCHAR(200)
);

-- Bien (normalizado): datos del empleado en su propia tabla
CREATE TABLE Empleados (
    Id INT IDENTITY PRIMARY KEY,
    Nombre NVARCHAR(100),
    Email NVARCHAR(255)
);

CREATE TABLE Inscripciones (
    Id INT IDENTITY PRIMARY KEY,
    EmpleadoId INT REFERENCES Empleados(Id),
    CursoId INT REFERENCES Cursos(Id)
);
```

## Como se usa en un proyecto real

1. Se diseña el esquema de StaffTrainingPlatform (o cualquier proyecto .NET) apuntando a 3FN como default, aplicando exactamente el mismo criterio ya estudiado en Postgres.
2. Se identifican dependencias transitivas revisando si una columna no clave depende de otra columna no clave en vez de la clave primaria.
3. Se documenta cualquier desnormalizacion deliberada con la misma justificacion de tradeoffs ya vista en el libro complementario.
4. Se conecta esto con el diseño de entidades de Entity Framework Core (ver capitulo de SQL Server con .NET), donde cada tabla normalizada corresponde a una clase de entidad.
5. Se aprovecha que este conocimiento es completamente transferible entre Postgres y SQL Server, dado que la normalizacion es teoria relacional independiente del motor.

## Criterio junior

Un junior debe poder aplicar normalizacion basica a un esquema T-SQL, transfiriendo directamente lo ya aprendido en Postgres.

## Criterio mid

Un mid debe diseñar esquemas normalizados a 3FN para proyectos .NET reales, identificando dependencias transitivas.

## Criterio senior

Un senior debe decidir cuando desnormalizar deliberadamente en un esquema SQL Server, documentando esa decision con el mismo rigor ya aplicado en Postgres.

## Errores comunes

- Repetir datos que deberian vivir en una unica tabla, generando anomalias de actualizacion identicas a las ya vistas en Postgres.
- Tratar la normalizacion como un concepto nuevo a reaprender en vez de reconocer que es la misma teoria relacional ya dominada.
- No documentar desnormalizacion deliberada, perdiendo el contexto de esa decision con el tiempo.
- No conectar el esquema normalizado con las entidades de Entity Framework Core correspondientes.

## Practica

- [ ] Identifica un esquema no normalizado en T-SQL y redisenalo a 3FN.
- [ ] Explica que anomalia de actualizacion previene esa normalizacion.
- [ ] Diseña el esquema normalizado de un dominio real (por ejemplo, capacitaciones de StaffTrainingPlatform).
- [ ] Conecta ese esquema con las entidades C# equivalentes que usarias en Entity Framework Core.
- [ ] Compara el proceso de normalizacion en T-SQL con el ya aplicado en Postgres, confirmando que es identico.

## Preguntas de repaso

- Por que la normalizacion es un concepto identico entre Postgres y SQL Server, mas alla de la sintaxis?
- Que anomalia de actualizacion previene la tercera forma normal?
- Como conectarias un esquema normalizado en T-SQL con clases de entidad de Entity Framework Core?
- Cuando seria valido desnormalizar deliberadamente un esquema SQL Server?
