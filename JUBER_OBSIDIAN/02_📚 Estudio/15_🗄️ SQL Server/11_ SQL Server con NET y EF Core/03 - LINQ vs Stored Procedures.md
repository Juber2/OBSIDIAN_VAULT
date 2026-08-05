---
tags:
  - SQLServer
  - DotNet
  - SQLServer-DotNet
Creado: 2026-07-29
Relacionado:
  - SQL Server con NET y EF Core
  - SQL SERVER
---

# 03 - LINQ vs Stored Procedures

## Que vas a aprender

En este capitulo vas a aprender cuando usar **LINQ to Entities** (queries expresadas en C#, traducidas automaticamente a T-SQL por EF Core) frente a **Stored Procedures** invocados desde EF Core, una decision de diseño recurrente en cualquier proyecto .NET con SQL Server.

## Explicacion

LINQ to Entities permite escribir queries en C# type-safe (`context.Cursos.Where(c => c.DuracionHoras > 10)`), que EF Core traduce automaticamente a T-SQL en tiempo de ejecucion. Esto ofrece gran productividad, refactorizacion segura (el compilador detecta errores de tipos), y queries que se adaptan naturalmente al modelo de objetos, pero para queries muy complejas (multiples joins con logica condicional compleja, agregaciones avanzadas) la traduccion automatica puede generar SQL suboptimo, un problema donde a veces vale la pena revisar el SQL generado con el mismo criterio ya aplicado con Execution Plans en el capitulo de indices.

Los **Stored Procedures** siguen siendo relevantes en escenarios especificos: operaciones batch con logica compleja mejor expresada directamente en T-SQL, cuando el equipo de DBA controla y optimiza queries criticas independientemente del ciclo de despliegue de la aplicacion, o cuando se necesita el maximo control sobre el plan de ejecucion. EF Core soporta invocar Stored Procedures directamente (`FromSqlRaw`/`FromSqlInterpolated` para lecturas, `ExecuteSqlRaw` para comandos), sin necesidad de abandonar EF Core por completo para casos puntuales donde LINQ no es la herramienta apropiada.

## Modelo mental

Pensa LINQ to Entities como pedirle a un traductor experto (EF Core) que convierta tus instrucciones en español (C#) a un idioma tecnico (T-SQL) automaticamente: rapido y comodo para la mayoria de conversaciones, pero para negociaciones muy tecnicas y especificas (queries complejas), a veces es mejor hablar directamente en el idioma tecnico vos mismo (Stored Procedures o SQL crudo) para asegurar que el mensaje se transmita exactamente como queres.

## Ejemplo base

```csharp
// LINQ to Entities — traducido automaticamente a T-SQL
var cursosLargos = await context.Cursos
    .Where(c => c.DuracionHoras > 10)
    .OrderBy(c => c.Nombre)
    .ToListAsync();

// Invocar un Stored Procedure existente desde EF Core
var resultado = await context.Cursos
    .FromSqlInterpolated($"EXEC dbo.ObtenerCursosPorCategoria {categoriaId}")
    .ToListAsync();

// Comando directo (INSERT/UPDATE/DELETE fuera del change tracker)
await context.Database.ExecuteSqlInterpolatedAsync(
    $"UPDATE Cursos SET Activo = 0 WHERE FechaVencimiento < {DateTime.Now}");
```

## Como se usa en un proyecto real

1. Se usa LINQ to Entities como enfoque predeterminado en StaffTrainingPlatform para la mayoria de queries, aprovechando la productividad y seguridad de tipos que ofrece.
2. Se revisa el SQL generado (con `ToQueryString()` o el logging de EF Core) para queries LINQ complejas, aplicando el mismo criterio de Execution Plans ya visto en el capitulo de indices cuando el rendimiento es sospechoso.
3. Se recurre a Stored Procedures o `FromSqlInterpolated` unicamente para casos puntuales donde LINQ genera SQL claramente suboptimo o donde se necesita control total sobre el plan de ejecucion.
4. Se usa siempre `FromSqlInterpolated`/parametros (nunca `FromSqlRaw` con concatenacion de strings) para evitar SQL injection, el mismo principio ya insistido en el tema de Dynamic SQL.
5. Se documenta la decision cuando se elige un Stored Procedure sobre LINQ, dejando clara la razon (performance, control del DBA, logica compleja) para futuros mantenedores.

## Criterio junior

Un junior debe poder escribir queries LINQ basicas y entender que se traducen automaticamente a T-SQL.

## Criterio mid

Un mid debe revisar el SQL generado por una query LINQ compleja e identificar cuando es suboptimo.

## Criterio senior

Un senior debe decidir con criterio cuando usar Stored Procedures sobre LINQ, balanceando productividad, mantenibilidad, y performance real medida.

## Errores comunes

- Usar `FromSqlRaw` con concatenacion de strings en vez de `FromSqlInterpolated` o parametros, exponiendo la aplicacion a SQL injection.
- Escribir queries LINQ excesivamente complejas que generan SQL ineficiente, sin revisar nunca el SQL real generado.
- Migrar a Stored Procedures prematuramente sin medir primero si LINQ realmente genera un problema de performance real.
- No documentar por que se eligio un Stored Procedure sobre LINQ, dejando la decision sin contexto para el equipo futuro.

## Practica

- [ ] Escribe una query LINQ compleja y revisa el SQL generado con `ToQueryString()`.
- [ ] Invoca un Stored Procedure existente desde EF Core con `FromSqlInterpolated`.
- [ ] Identifica un caso donde LINQ generaria SQL suboptimo comparado con un Stored Procedure escrito a mano.
- [ ] Compara el SQL generado por LINQ con el Execution Plan ya estudiado en el capitulo de indices.
- [ ] Documenta un criterio propio de cuando usarias Stored Procedures sobre LINQ en un proyecto real.

## Preguntas de repaso

- Que ventaja ofrece LINQ to Entities sobre escribir SQL directamente?
- En que escenario seguiria siendo apropiado usar un Stored Procedure en vez de LINQ?
- Por que `FromSqlInterpolated` es mas seguro que `FromSqlRaw` con concatenacion?
- Como revisarias si una query LINQ esta generando SQL suboptimo?
