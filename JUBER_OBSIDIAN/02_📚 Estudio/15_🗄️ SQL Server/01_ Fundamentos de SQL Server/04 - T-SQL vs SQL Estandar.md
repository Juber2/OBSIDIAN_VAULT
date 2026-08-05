---
tags:
  - SQLServer
  - Bases-de-Datos
  - SQLServer-Fundamentos
Creado: 2026-07-29
Relacionado:
  - Fundamentos de SQL Server
  - SQL SERVER
---

# 04 - T-SQL vs SQL Estandar

## Que vas a aprender

En este capitulo vas a entender **T-SQL** (Transact-SQL), el dialecto especifico de SQL Server, y sus diferencias practicas de sintaxis frente al SQL estandar (y frente a Postgres, ver libro complementario) que vas a encontrar constantemente al escribir queries reales.

## Explicacion

T-SQL extiende el estandar SQL con sintaxis propia de Microsoft: `TOP N` para limitar resultados (en vez de `LIMIT` de Postgres/MySQL), corchetes `[nombre]` para identificadores con espacios o palabras reservadas (en vez de comillas dobles del estandar), `GETDATE()` para la fecha actual (en vez de `now()` de Postgres), concatenacion con `+` para strings (en vez de `||` del estandar SQL usado por Postgres), y `IDENTITY` para columnas autoincrementales (el equivalente directo del `GENERATED ALWAYS AS IDENTITY` ya visto en el libro de Postgres).

Estas diferencias de sintaxis son la primera fuente de confusion al moverse entre motores, especialmente relevante si trabajas con ambos (Postgres en NestJS, SQL Server en StaffTrainingPlatform con .NET). Conocer las equivalencias directas acelera significativamente la traduccion mental entre ambos dialectos.

## Modelo mental

Pensa T-SQL y el SQL de Postgres como dos dialectos del mismo idioma base (SQL): un hablante de un dialecto puede entender al otro con algo de esfuerzo, pero ciertas palabras y expresiones especificas (`TOP` vs `LIMIT`, `GETDATE()` vs `now()`) simplemente no se traducen literalmente palabra por palabra, hay que aprender la expresion equivalente correcta en cada dialecto.

## Ejemplo base

```sql
-- T-SQL (SQL Server)
SELECT TOP 10 * FROM Pedidos ORDER BY CreadoEn DESC;
SELECT GETDATE();
SELECT Nombre + ' ' + Apellido AS NombreCompleto FROM Clientes;
CREATE TABLE Usuarios (Id INT IDENTITY(1,1) PRIMARY KEY, Email NVARCHAR(255));

-- Equivalente en Postgres (ver libro complementario)
SELECT * FROM pedidos ORDER BY creado_en DESC LIMIT 10;
SELECT now();
SELECT nombre || ' ' || apellido AS nombre_completo FROM clientes;
CREATE TABLE usuarios (id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY, email text);
```

## Como se usa en un proyecto real

1. Se usa `TOP N` para limitar resultados en T-SQL, recordando que la sintaxis difiere de `LIMIT` si se trabaja tambien con Postgres.
2. Se usa `NVARCHAR` (soporte Unicode) en vez de `VARCHAR` para texto que puede contener caracteres internacionales, una convencion especifica de SQL Server relevante para datos como nombres de usuarios.
3. Se documenta el diccionario de equivalencias T-SQL/Postgres para el equipo que trabaja con ambos motores (NestJS con Postgres, StaffTrainingPlatform con SQL Server).
4. Se aprovecha `IDENTITY` de la misma forma que `GENERATED ALWAYS AS IDENTITY` de Postgres, aplicando el mismo criterio de diseño ya visto en ese libro.
5. Se evita escribir SQL "generico" asumiendo portabilidad automatica entre motores sin verificar la sintaxis especifica de cada uno.

## Criterio junior

Un junior debe poder escribir consultas T-SQL basicas usando `TOP`, `GETDATE()`, y corchetes para identificadores especiales.

## Criterio mid

Un mid debe traducir mentalmente entre T-SQL y SQL estandar/Postgres al trabajar en proyectos que usan ambos motores.

## Criterio senior

Un senior debe documentar las diferencias de dialecto relevantes para el equipo, y decidir cuando escribir SQL "portable" versus aprovechar sintaxis especifica de cada motor segun el contexto.

## Errores comunes

- Intentar usar `LIMIT` en SQL Server esperando que funcione igual que en Postgres, sin saber que la sintaxis correcta es `TOP`.
- Usar `VARCHAR` para texto que puede contener caracteres internacionales, cuando `NVARCHAR` seria la eleccion correcta en SQL Server.
- Asumir portabilidad automatica de SQL entre Postgres y SQL Server sin verificar diferencias reales de sintaxis.
- No documentar el diccionario de equivalencias para un equipo que trabaja con ambos motores simultaneamente.

## Practica

- [ ] Escribe una consulta T-SQL basica usando `TOP`, `GETDATE()` y concatenacion con `+`.
- [ ] Traduce esa misma consulta al equivalente de Postgres.
- [ ] Compara `VARCHAR` y `NVARCHAR` y decide cuando usar cada uno.
- [ ] Crea una tabla con `IDENTITY` y compara con el `GENERATED ALWAYS AS IDENTITY` de Postgres.
- [ ] Documenta un diccionario de equivalencias T-SQL/Postgres para tu propio uso.

## Preguntas de repaso

- Que diferencia de sintaxis hay entre `TOP N` de T-SQL y `LIMIT` de Postgres?
- Por que `NVARCHAR` es preferible a `VARCHAR` para texto que puede contener caracteres internacionales?
- Como se concatenan strings en T-SQL, y en que se diferencia de Postgres?
- Que equivalente directo tiene `IDENTITY` de T-SQL en el Postgres ya estudiado?
