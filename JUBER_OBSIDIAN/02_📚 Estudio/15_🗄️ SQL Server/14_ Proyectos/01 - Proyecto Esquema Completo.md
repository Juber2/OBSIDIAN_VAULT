---
tags:
  - SQLServer
  - Proyectos
  - SQLServer-Proyectos
Creado: 2026-07-29
Relacionado:
  - Proyectos
  - SQL SERVER
---

# 01 - Proyecto Esquema Completo

## Que vas a aprender

En este proyecto vas a aplicar los fundamentos de los primeros capitulos (Fundamentos, T-SQL, Modelado, Indices) diseñando e implementando un esquema completo y normalizado desde cero para un dominio realista.

## Explicacion

Este es el primer proyecto integrador del libro, enfocado en aplicar correctamente normalizacion (hasta 3FN salvo denormalizacion consciente y justificada), tipos de datos apropiados para cada columna, claves primarias y foraneas con las restricciones de integridad correctas, e indices basicos sobre las columnas que las queries tipicas del dominio realmente necesitan.

El dominio sugerido es un sistema de gestion de cursos y capacitaciones (deliberadamente cercano a StaffTrainingPlatform), con entidades como Cursos, Modulos, Estudiantes, Inscripciones, y Progreso, permitiendo aplicar el conocimiento en un contexto que ya te resulta familiar del proyecto real.

## Modelo mental

Pensa este proyecto como construir los cimientos de un edificio: cada decision de modelado (tipos de datos, relaciones, restricciones) que tomes aca sera la base sobre la que los proyectos siguientes (optimizacion, alta disponibilidad, integracion con .NET) se construiran, igual que un cimiento mal diseñado limita todo lo que se construye encima.

## Ejemplo base

```sql
CREATE TABLE Cursos (
    Id INT IDENTITY PRIMARY KEY,
    Nombre NVARCHAR(200) NOT NULL,
    DuracionHoras INT NOT NULL CHECK (DuracionHoras > 0),
    Activo BIT NOT NULL DEFAULT 1
);

CREATE TABLE Estudiantes (
    Id INT IDENTITY PRIMARY KEY,
    Email NVARCHAR(256) NOT NULL UNIQUE,
    Nombre NVARCHAR(200) NOT NULL
);

CREATE TABLE Inscripciones (
    Id INT IDENTITY PRIMARY KEY,
    EstudianteId INT NOT NULL FOREIGN KEY REFERENCES Estudiantes(Id),
    CursoId INT NOT NULL FOREIGN KEY REFERENCES Cursos(Id),
    FechaInscripcion DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT UQ_EstudianteCurso UNIQUE (EstudianteId, CursoId)
);

CREATE INDEX IX_Inscripciones_CursoId ON Inscripciones(CursoId);
```

## Como se usa en un proyecto real

1. Se diseña el esquema completo (Cursos, Modulos, Estudiantes, Inscripciones, Progreso) aplicando normalizacion hasta 3FN, documentando cualquier denormalizacion consciente.
2. Se eligen tipos de datos apropiados para cada columna (evitando `NVARCHAR(MAX)` por defecto, usando `BIT` para booleanos, `DATETIME2` para fechas), aplicando lo aprendido en el capitulo de Fundamentos.
3. Se agregan restricciones de integridad (`CHECK`, `UNIQUE`, `FOREIGN KEY`) que reflejen las reglas de negocio reales del dominio, no solo las minimas necesarias.
4. Se crean indices basicos sobre columnas de foreign key y sobre las columnas mas usadas en filtros, verificando con el Execution Plan que efectivamente se usan.
5. Se compara el esquema resultante con el de StaffTrainingPlatform real (si es accesible), identificando decisiones de diseño similares o distintas.

## Criterio de exito

- El esquema esta normalizado hasta 3FN, con cualquier denormalizacion documentada y justificada.
- Cada columna usa el tipo de dato mas apropiado, no el mas generico disponible.
- Las relaciones tienen restricciones de integridad reales, no solo llaves foraneas sin `CHECK` ni `UNIQUE` donde corresponderia.
- Existen indices basicos sobre columnas de join y filtro frecuente, verificados con `SET STATISTICS IO ON` o el Execution Plan.

## Errores comunes

- Sobre-normalizar hasta el punto de necesitar demasiados joins para queries simples y frecuentes, sin evaluar el trade-off practico.
- Usar `NVARCHAR(MAX)` por defecto en todas las columnas de texto, ignorando el tipo mas apropiado segun el uso real.
- Olvidar restricciones `UNIQUE`/`CHECK` que reflejarian reglas de negocio reales del dominio.
- No verificar con el Execution Plan que los indices creados realmente se usan en las queries tipicas del dominio.

## Practica

- [ ] Diseña el modelo entidad-relacion completo antes de escribir el DDL.
- [ ] Implementa el esquema completo con tipos de datos y restricciones apropiadas.
- [ ] Crea indices basicos y verifica su uso con el Execution Plan de queries tipicas.
- [ ] Documenta cualquier decision de denormalizacion consciente que hayas tomado.
- [ ] Compara tu esquema resultante con el diseño real de StaffTrainingPlatform.

## Preguntas de repaso

- Que decisiones de normalizacion tomaste, y por que?
- Que tipos de datos elegiste para cada columna, y que alternativa descartaste?
- Que indices creaste, y como verificaste que se usan realmente?
- Que harias diferente si este esquema tuviera que escalar a millones de registros?
