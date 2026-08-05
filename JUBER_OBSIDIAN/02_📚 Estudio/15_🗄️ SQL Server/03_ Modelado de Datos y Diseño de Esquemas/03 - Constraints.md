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

# 03 - Constraints

## Que vas a aprender

En este capitulo vas a aplicar `NOT NULL`, `UNIQUE`, `CHECK` en T-SQL, exactamente los mismos conceptos ya estudiados en Postgres, con la misma justificacion de defensa en profundidad ya vista tanto en ese libro como en el capitulo de Seguridad Senior de NestJS.

## Explicacion

La sintaxis de constraints en T-SQL es practicamente identica a Postgres: `NOT NULL`, `UNIQUE` (con soporte multi-columna), y `CHECK (condicion)` para reglas de negocio arbitrarias. La razon de valor es identica: son la ultima linea de defensa contra datos invalidos, sin importar si entran via la API de .NET, un script de migracion, o acceso directo a la base de datos, complementando (no reemplazando) la validacion ya presente en la capa de aplicacion con Data Annotations o Fluent Validation de .NET.

Una particularidad de T-SQL: se pueden nombrar constraints explicitamente con `CONSTRAINT nombre` antes de la definicion, la misma practica recomendada ya vista en Postgres para generar mensajes de error legibles.

## Modelo mental

El modelo mental es identico: los constraints son el control de calidad final de una fabrica, verificando cada dato que intenta entrar sin importar por que puerta (camino de codigo) intento pasar, incluso si la validacion de la aplicacion (.NET) ya lo revisa antes.

## Ejemplo base

```sql
CREATE TABLE Cursos (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    Codigo NVARCHAR(20) NOT NULL,
    Nombre NVARCHAR(200) NOT NULL,
    CupoMaximo INT NOT NULL,
    CONSTRAINT UQ_Cursos_Codigo UNIQUE (Codigo),
    CONSTRAINT CHK_Cursos_CupoPositivo CHECK (CupoMaximo > 0)
);

CREATE TABLE Reservas (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    SalaId BIGINT NOT NULL,
    Fecha DATE NOT NULL,
    CONSTRAINT UQ_Reservas_SalaFecha UNIQUE (SalaId, Fecha)
);
```

## Como se usa en un proyecto real

1. Se replica en SQL Server toda regla de negocio critica que ya existe en la validacion de .NET (Data Annotations, FluentValidation), aplicando la misma defensa en profundidad ya vista en Postgres.
2. Se usan constraints `UNIQUE` multi-columna para reglas de exclusividad, identico al criterio ya establecido.
3. Se nombran explicitamente los constraints con `CONSTRAINT nombre`, aplicando la misma buena practica ya recomendada en Postgres.
4. Se prueba que un constraint efectivamente rechaza datos invalidos antes de confiar en el.
5. Se documenta que reglas de negocio de StaffTrainingPlatform deberian replicarse como constraints de base de datos.

## Criterio junior

Un junior debe poder agregar `NOT NULL`, `UNIQUE` y `CHECK` basicos en T-SQL, transfiriendo lo ya aprendido en Postgres.

## Criterio mid

Un mid debe usar constraints multi-columna y nombrarlos de forma descriptiva.

## Criterio senior

Un senior debe definir que reglas de negocio deben replicarse como constraints en un proyecto .NET, coordinando con las validaciones ya existentes en la capa de aplicacion.

## Errores comunes

- Confiar exclusivamente en validacion de .NET sin replicar reglas criticas como constraints de base de datos.
- No nombrar constraints explicitamente, generando mensajes de error crípticos.
- No coordinar las reglas de constraint con las validaciones ya existentes de Data Annotations/FluentValidation, generando duplicacion inconsistente.
- No probar que un constraint realmente rechaza el dato invalido esperado.

## Practica

- [ ] Agrega `NOT NULL`, `UNIQUE` y `CHECK` nombrados explicitamente a una tabla T-SQL.
- [ ] Crea un constraint `UNIQUE` multi-columna para una regla de exclusividad real.
- [ ] Provoca la violacion de cada constraint y observa el mensaje de error.
- [ ] Identifica una regla de negocio de StaffTrainingPlatform que deberia replicarse como constraint.
- [ ] Compara la sintaxis de constraints de T-SQL con la ya estudiada en Postgres.

## Preguntas de repaso

- Por que vale la pena replicar reglas de negocio en constraints de base de datos si ya existen en la validacion de .NET?
- Como se nombra explicitamente un constraint en T-SQL?
- Que ventaja da un constraint `UNIQUE` multi-columna sobre validar esa exclusividad solo en la aplicacion?
- Como coordinarias constraints de base de datos con Data Annotations de Entity Framework?
