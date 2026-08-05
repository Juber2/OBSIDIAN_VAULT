---
tags:
  - SQLServer
  - TSQL
  - SQLServer-TSQL
Creado: 2026-07-29
Relacionado:
  - T-SQL Fundamentals
  - SQL SERVER
---

# 01 - CREATE TABLE y Tipos de Datos

## Que vas a aprender

En este capitulo vas a aprender a crear tablas con `CREATE TABLE` en T-SQL, y los tipos de datos fundamentales de SQL Server: numericos, texto (`VARCHAR`/`NVARCHAR`), fecha/hora, y `IDENTITY` para autoincrementales.

## Explicacion

`CREATE TABLE` en T-SQL sigue una sintaxis similar al estandar, con tipos especificos de SQL Server: `INT` (32 bits), `BIGINT` (64 bits, recomendado para IDs de tablas grandes, el mismo criterio ya visto en Postgres), `DECIMAL(precision, escala)` para valores monetarios exactos (equivalente a `numeric` de Postgres, nunca `FLOAT` para dinero por las mismas razones de imprecision ya explicadas en ese libro).

Para texto, la eleccion entre `VARCHAR` (ASCII, mas liviano) y `NVARCHAR` (Unicode, soporta cualquier idioma, ocupa el doble de espacio por caracter) es una decision real: `NVARCHAR` es la eleccion segura por defecto para cualquier texto que pueda contener nombres de usuarios, direcciones, o contenido internacional. Para fechas, `DATETIME2` (mayor precision y rango que el legacy `DATETIME`) es la eleccion moderna recomendada, y `IDENTITY(1,1)` genera valores autoincrementales, el equivalente directo de `GENERATED ALWAYS AS IDENTITY` de Postgres.

## Modelo mental

Pensa `CREATE TABLE` en T-SQL como diseñar el mismo tipo de formulario oficial ya visto conceptualmente en Postgres, pero con el vocabulario especifico de Microsoft: donde Postgres dice `text`, SQL Server distingue entre `VARCHAR` (solo caracteres basicos) y `NVARCHAR` (cualquier idioma del mundo), un matiz que Postgres no necesita porque `text` ya soporta Unicode de forma nativa siempre.

## Ejemplo base

```sql
CREATE TABLE Usuarios (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    Email NVARCHAR(255) NOT NULL,
    Nombre NVARCHAR(100) NOT NULL,
    Edad INT,
    Saldo DECIMAL(10, 2) DEFAULT 0,
    Activo BIT DEFAULT 1,          -- BIT es el booleano de SQL Server (0 o 1)
    CreadoEn DATETIME2 DEFAULT SYSDATETIME()
);
```

## Como se usa en un proyecto real

1. Se usa `BIGINT IDENTITY(1,1)` para IDs autoincrementales en tablas nuevas, aplicando el mismo criterio de rango ya visto para `bigint` en Postgres.
2. Se usa `DECIMAL` (nunca `FLOAT`) para cualquier valor monetario, evitando errores de redondeo acumulados.
3. Se usa `NVARCHAR` por default para texto que puede contener contenido internacional (nombres, direcciones), reservando `VARCHAR` solo para datos garantizadamente ASCII (codigos internos, por ejemplo).
4. Se usa `DATETIME2` en vez del legacy `DATETIME`, aprovechando mayor precision y rango de fechas soportado.
5. Se conecta esto con Entity Framework Core (ver capitulo de SQL Server con .NET), donde estos tipos mapean directamente a propiedades de una clase de entidad en C#.

## Criterio junior

Un junior debe poder crear una tabla simple con columnas de tipos basicos usando la sintaxis de T-SQL.

## Criterio mid

Un mid debe elegir correctamente entre `VARCHAR`/`NVARCHAR` segun el contenido esperado, y usar `DATETIME2` en vez del legacy `DATETIME`.

## Criterio senior

Un senior debe definir convenciones de tipos de datos para todo el equipo (cuando usar NVARCHAR, tamaños de DECIMAL para dinero), documentando la justificacion.

## Errores comunes

- Usar `VARCHAR` para texto que puede contener caracteres internacionales, causando corrupcion o perdida de datos con nombres o contenido no-ASCII.
- Usar `FLOAT` para valores monetarios, arriesgando errores de redondeo acumulados con el tiempo.
- Usar el legacy `DATETIME` en vez de `DATETIME2`, perdiendo precision y rango disponible en versiones modernas.
- No dimensionar apropiadamente `DECIMAL(precision, escala)` segun el rango real de valores esperados.

## Practica

- [ ] Crea una tabla con columnas de tipos numericos, texto, fecha y booleano en T-SQL.
- [ ] Compara `VARCHAR` y `NVARCHAR` insertando texto con caracteres especiales en cada uno.
- [ ] Crea una columna monetaria con `DECIMAL` y verifica que no pierde precision.
- [ ] Usa `DATETIME2` y compara su precision con el legacy `DATETIME`.
- [ ] Conecta el tipo de cada columna con la propiedad C# equivalente que usarias en Entity Framework Core.

## Preguntas de repaso

- Que diferencia hay entre `VARCHAR` y `NVARCHAR`, y cuando usarias cada uno?
- Por que `DECIMAL` es el tipo correcto para valores monetarios y no `FLOAT`?
- Que ventaja tiene `DATETIME2` sobre el legacy `DATETIME`?
- Como se relaciona `IDENTITY(1,1)` con `GENERATED ALWAYS AS IDENTITY` de Postgres?
