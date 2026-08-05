---
tags:
  - SQLServer
  - TSQL-Avanzado
  - SQLServer-Avanzado
Creado: 2026-07-29
Relacionado:
  - T-SQL Avanzado
  - SQL SERVER
---

# 05 - Dynamic SQL

## Que vas a aprender

En este capitulo vas a cerrar el capitulo aprendiendo **Dynamic SQL** en T-SQL (`sp_executesql`), construccion de queries en tiempo de ejecucion, y el riesgo critico de **SQL injection** que conecta directamente con el capitulo de Seguridad Senior del libro de NestJS.

## Explicacion

Dynamic SQL permite construir y ejecutar una sentencia SQL como texto en tiempo de ejecucion, util cuando la estructura de la consulta (no solo sus valores) varia segun condiciones (por ejemplo, ordenar por una columna elegida dinamicamente por el usuario, algo que no se puede parametrizar de forma normal). `sp_executesql` es la forma **correcta y segura** de ejecutar Dynamic SQL, dado que soporta **parametros reales** (a diferencia de `EXEC` con concatenacion directa de strings), previniendo SQL injection de la misma forma que los parametros ya vistos en `.query()` de TypeORM (ver libro de NestJS) o `$queryRaw` de Prisma (ver libro de Postgres).

El error de seguridad mas grave y comun es **concatenar directamente valores de usuario** dentro del texto de la consulta dinamica sin parametrizar, exactamente el mismo riesgo de inyeccion SQL ya advertido consistentemente en los libros de NestJS y Postgres, ahora aplicado a T-SQL con su propia sintaxis especifica.

## Modelo mental

Pensa Dynamic SQL como escribir una carta cuya estructura general cambia segun la ocasion (a veces menciona tres temas, a veces cinco), pero cuyo contenido variable especifico (nombres, cantidades) siempre debe insertarse a traves de un mecanismo seguro y controlado (parametros), nunca copiando y pegando directamente lo que alguien te dictó sin verificar, porque esa persona podria estar dictandote instrucciones maliciosas disfrazadas de datos.

## Ejemplo base

```sql
-- MAL: concatenacion directa, vulnerable a SQL injection
DECLARE @ColumnaOrden NVARCHAR(50) = 'CreadoEn';
DECLARE @SQL NVARCHAR(MAX) = 'SELECT * FROM Pedidos ORDER BY ' + @ColumnaOrden;
EXEC (@SQL);   -- si @ColumnaOrden viniera de input de usuario sin validar, es vulnerable

-- BIEN: sp_executesql con parametros reales para VALORES
DECLARE @ClienteId BIGINT = 42;
DECLARE @SQL NVARCHAR(MAX) = N'SELECT * FROM Pedidos WHERE ClienteId = @ClienteId';
EXEC sp_executesql @SQL, N'@ClienteId BIGINT', @ClienteId = @ClienteId;

-- Para nombres de columna/tabla dinamicos (que NO se pueden parametrizar), validar contra una whitelist
DECLARE @ColumnasPermitidas TABLE (Nombre NVARCHAR(50));
INSERT INTO @ColumnasPermitidas VALUES ('CreadoEn'), ('Total');
IF EXISTS (SELECT 1 FROM @ColumnasPermitidas WHERE Nombre = @ColumnaOrden)
BEGIN
    -- solo entonces se arma el Dynamic SQL con ese nombre de columna verificado
END
```

## Como se usa en un proyecto real

1. Se usa `sp_executesql` con parametros reales para cualquier valor variable dentro de Dynamic SQL, nunca concatenacion directa de strings.
2. Se valida cualquier nombre de columna/tabla dinamico (que no puede parametrizarse como un valor normal) contra una **whitelist explicita** de nombres permitidos, el mismo patron ya visto para paginacion dinamica en el libro de NestJS.
3. Se reserva Dynamic SQL para casos genuinamente necesarios (estructura de query variable), no como atajo habitual cuando SQL normal ya resolveria el caso.
4. Se conecta esto directamente con el capitulo de Seguridad Senior del libro de NestJS y el capitulo de Repository Pattern del libro de Postgres, aplicando el mismo principio de parametrizacion segura en un tercer contexto (T-SQL).
5. Se revisa en code review cualquier uso de Dynamic SQL, verificando explicitamente que no hay concatenacion directa de input de usuario sin parametrizar o sin whitelist.

## Criterio junior

Un junior debe entender por que concatenar directamente valores en Dynamic SQL es peligroso, y usar `sp_executesql` con parametros.

## Criterio mid

Un mid debe validar nombres de columna/tabla dinamicos contra una whitelist cuando no pueden parametrizarse normalmente.

## Criterio senior

Un senior debe revisar en code review cualquier uso de Dynamic SQL en el equipo, estableciendo como estandar obligatorio el uso de `sp_executesql` parametrizado.

## Errores comunes

- Concatenar directamente valores de usuario en Dynamic SQL sin parametrizar, el error de seguridad mas grave y comun en este tema.
- No validar nombres de columna/tabla dinamicos contra una whitelist, permitiendo que un input malicioso especifique un nombre de objeto inesperado.
- Usar Dynamic SQL como atajo habitual cuando SQL normal parametrizado ya resolveria el caso sin la complejidad adicional.
- No revisar en code review el uso de Dynamic SQL con el mismo rigor de seguridad ya aplicado a SQL crudo en NestJS/Postgres.

## Practica

- [ ] Escribe un ejemplo de Dynamic SQL vulnerable con concatenacion directa, y explica el riesgo.
- [ ] Reescribe ese mismo caso usando `sp_executesql` con parametros reales.
- [ ] Implementa una whitelist para validar un nombre de columna dinamico usado en ordenamiento.
- [ ] Conecta este capitulo con el mismo principio de seguridad ya visto en NestJS (capitulo de Seguridad Senior) y Postgres (Repository Pattern).
- [ ] Revisa (o simula) un code review de Dynamic SQL aplicando este criterio de seguridad.

## Preguntas de repaso

- Por que concatenar directamente valores de usuario en Dynamic SQL es un riesgo critico de seguridad?
- Como usarias `sp_executesql` para parametrizar de forma segura un valor variable?
- Que estrategia usarias para un nombre de columna/tabla dinamico que no puede parametrizarse normalmente?
- Como se conecta este riesgo con el mismo principio de seguridad ya visto en NestJS y Postgres?

## Cierre del capitulo

Con este tema cierras el capitulo de T-SQL avanzado, habiendo dominado CTEs, funciones de ventana, JSON (con sus limitaciones frente a Postgres), MERGE, y Dynamic SQL seguro, todo conectado consistentemente con el conocimiento ya solido de SQL construido en el libro de PostgreSQL.
