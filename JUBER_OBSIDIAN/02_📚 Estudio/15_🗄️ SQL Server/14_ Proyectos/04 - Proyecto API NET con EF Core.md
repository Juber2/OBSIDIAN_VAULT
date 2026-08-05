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

# 04 - Proyecto API .NET con EF Core

## Que vas a aprender

En este proyecto vas a aplicar el capitulo de SQL Server con .NET y EF Core construyendo una API .NET minima completa sobre el esquema de los proyectos anteriores, con Migrations, testing con Testcontainers, y una decision documentada sobre LINQ vs Stored Procedures.

## Explicacion

Este proyecto integra todo lo aprendido sobre la conexion entre .NET y SQL Server en un caso de uso completo y realista: una API .NET (Minimal API o controladores tradicionales) que expone endpoints CRUD sobre Cursos e Inscripciones (el esquema ya diseñado en el primer proyecto), usando Entity Framework Core Code First con Migrations versionadas, aplicando el patron de acceso a datos decidido conscientemente (con o sin Repository explicito, ver ese tema), y con al menos un endpoint que demuestre una decision consciente entre LINQ y un Stored Procedure invocado directamente.

El componente de testing es central: se espera una suite de tests de integracion usando Testcontainers con SQL Server real (no el In-Memory Provider), validando que el comportamiento observado en tests coincida genuinamente con el comportamiento de produccion, cerrando el circulo completo desde el diseño del esquema hasta una aplicacion funcional y bien testeada.

## Modelo mental

Pensa este proyecto como la culminacion practica de todo el libro: ya no es un ejercicio aislado sobre un concepto especifico, es construir el tipo de sistema real (una API .NET respaldada por SQL Server) que refleja directamente el trabajo que harias en StaffTrainingPlatform o un proyecto profesional equivalente.

## Ejemplo base

```csharp
// Program.cs — Minimal API con EF Core
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();

app.MapGet("/cursos", async (AppDbContext db) =>
    await db.Cursos.Where(c => c.Activo).ToListAsync());

app.MapPost("/cursos", async (AppDbContext db, Curso curso) =>
{
    db.Cursos.Add(curso);
    await db.SaveChangesAsync();
    return Results.Created($"/cursos/{curso.Id}", curso);
});

app.Run();
```

```csharp
// Test de integracion con Testcontainers y SQL Server real
public class CursosApiTests : IClassFixture<DatabaseFixture>
{
    [Fact]
    public async Task GetCursos_RetornaSoloActivos()
    {
        // Arrange: usar el DbContext conectado al contenedor real
        // Act: llamar al endpoint o metodo equivalente
        // Assert: verificar que solo se retornan cursos activos
    }
}
```

## Como se usa en un proyecto real

1. Se construye una API .NET completa (Minimal API o controladores) sobre el esquema diseñado en el primer proyecto, con endpoints CRUD para Cursos e Inscripciones.
2. Se generan y versionan Migrations de EF Core Code First, revisando cada una antes de aplicarla.
3. Se documenta la decision consciente tomada sobre Repository Pattern (con o sin capa explicita) para este proyecto especifico.
4. Se implementa al menos un endpoint que invoque un Stored Procedure directamente, documentando por que se eligio esa alternativa sobre LINQ en ese caso especifico.
5. Se construye una suite de tests de integracion con Testcontainers y SQL Server real, validando el comportamiento genuino de la API contra la base de datos.

## Criterio de exito

- La API expone endpoints CRUD funcionales sobre el esquema del primer proyecto, con Migrations de EF Core versionadas.
- Existe una decision documentada sobre Repository Pattern, justificada para el contexto de este proyecto.
- Al menos un endpoint invoca un Stored Procedure directamente, con la justificacion de esa eleccion documentada.
- La suite de tests usa Testcontainers con SQL Server real, no el In-Memory Provider, para validar comportamiento genuino.

## Errores comunes

- Usar el In-Memory Provider para los tests de integracion, perdiendo fidelidad al comportamiento real de SQL Server.
- No revisar las Migrations generadas automaticamente antes de aplicarlas.
- Agregar un Repository Pattern generico sin justificar su necesidad real para este proyecto especifico.
- No documentar por que se eligio LINQ o Stored Procedure para cada endpoint especifico.

## Practica

- [ ] Construye la API .NET completa con endpoints CRUD sobre el esquema del primer proyecto.
- [ ] Genera y aplica Migrations de EF Core Code First, revisando cada una.
- [ ] Documenta tu decision sobre Repository Pattern para este proyecto.
- [ ] Implementa al menos un endpoint que invoque un Stored Procedure directamente.
- [ ] Construye una suite de tests de integracion con Testcontainers y SQL Server real.

## Preguntas de repaso

- Que decision tomaste sobre Repository Pattern para este proyecto, y por que?
- En que endpoint usaste un Stored Procedure en vez de LINQ, y que justifico esa decision?
- Que ventaja obtuviste al testear con Testcontainers en vez del In-Memory Provider?
- Que cambiarias en el diseño de esta API si tuviera que escalar a produccion real?
