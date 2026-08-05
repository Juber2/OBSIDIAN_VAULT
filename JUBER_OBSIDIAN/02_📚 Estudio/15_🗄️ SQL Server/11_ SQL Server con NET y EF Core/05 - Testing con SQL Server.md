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

# 05 - Testing con SQL Server

## Que vas a cerrar

En este capitulo vas a cerrar el capitulo aprendiendo estrategias de testing para codigo que depende de SQL Server: **In-Memory Provider** (rapido pero con limitaciones), **SQLite in-memory** (alternativa intermedia), y **Testcontainers** con SQL Server real (la opcion mas fiel), aplicando directamente el patron de integration testing con contenedores ya visto en el libro de Docker.

## Explicacion

El **In-Memory Provider** de EF Core (`UseInMemoryDatabase`) permite tests rapidos sin ninguna base de datos real, pero tiene una limitacion critica: **no aplica las mismas restricciones y comportamientos que SQL Server real** (no valida constraints unicos de la misma forma, no traduce el mismo SQL, no refleja comportamiento de triggers/stored procedures), generando falsos positivos donde un test pasa en memoria pero el mismo codigo falla contra SQL Server real. Por esta razon, la comunidad .NET recomienda cada vez mas evitar el In-Memory Provider para tests que validan comportamiento real de la capa de datos.

**Testcontainers** (la libreria .NET equivalente a la ya usada conceptualmente en el libro de Docker para integration testing) permite levantar un contenedor real de SQL Server (la misma imagen oficial del capitulo de Docker) automaticamente al inicio de la suite de tests, ejecutar los tests contra una instancia real, y destruirlo al finalizar, garantizando que el comportamiento validado en tests coincida exactamente con produccion, el mismo principio de "no mockear la base de datos en tests de integracion" ya reforzado en experiencias previas con Postgres.

## Modelo mental

Pensa el In-Memory Provider como practicar manejo en un simulador de auto: util para aprender los controles basicos rapidamente, pero un simulador no replica exactamente como se comporta un auto real en una calle con baches, trafico, y clima real. Testcontainers es como practicar en un auto real pero en un circuito cerrado seguro: mas lento de preparar, pero el comportamiento que observas es genuinamente el mismo que tendrias en la calle real (produccion).

## Ejemplo base

```csharp
// In-Memory Provider — rapido pero con limitaciones reales
services.AddDbContext<AppDbContext>(options =>
    options.UseInMemoryDatabase("TestDb"));

// Testcontainers — SQL Server real en un contenedor Docker
public class DatabaseFixture : IAsyncLifetime
{
    private readonly MsSqlContainer _container = new MsSqlBuilder().Build();

    public async Task InitializeAsync() => await _container.StartAsync();
    public async Task DisposeAsync() => await _container.DisposeAsync();

    public string ConnectionString => _container.GetConnectionString();
}

public class CursoTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public CursoTests(DatabaseFixture fixture) => _fixture = fixture;

    [Fact]
    public async Task CrearCurso_PersisteCorrectamente()
    {
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseSqlServer(_fixture.ConnectionString)
            .Options;
        await using var context = new AppDbContext(options);
        await context.Database.MigrateAsync();

        context.Cursos.Add(new Curso { Nombre = "Test" });
        await context.SaveChangesAsync();

        Assert.Equal(1, await context.Cursos.CountAsync());
    }
}
```

## Como se usa en un proyecto real

1. Se usa Testcontainers con la imagen oficial de SQL Server (ya vista en el capitulo de Docker) para tests de integracion de StaffTrainingPlatform que validan comportamiento real de la capa de datos.
2. Se evita el In-Memory Provider para tests que validan constraints, comportamiento de queries complejas, o cualquier logica donde la fidelidad al motor real importa.
3. Se aplica `MigrateAsync()` al inicio de cada suite de tests con Testcontainers, garantizando que el esquema de test coincida exactamente con las Migrations reales del proyecto.
4. Se conecta esto directamente con el principio ya reforzado de "integration tests deben usar una base de datos real, no mocks", aplicando el mismo criterio ya usado con Postgres en otros contextos.
5. Se acepta el costo de tiempo adicional de Testcontainers (levantar un contenedor real) como el precio necesario para tener confianza genuina en que los tests reflejan el comportamiento de produccion.

## Criterio junior

Un junior debe poder escribir un test usando el In-Memory Provider y entender sus limitaciones basicas.

## Criterio mid

Un mid debe configurar Testcontainers con SQL Server real para tests de integracion que requieren fidelidad al comportamiento de produccion.

## Criterio senior

Un senior debe definir la estrategia de testing completa del equipo, decidiendo cuando el In-Memory Provider es aceptable (tests unitarios rapidos sin logica de datos critica) y cuando Testcontainers es obligatorio (integration tests).

## Errores comunes

- Usar el In-Memory Provider para validar constraints unicos o comportamiento especifico de SQL Server, obteniendo falsos positivos que fallan en produccion real.
- No aplicar Migrations reales en tests con Testcontainers, dejando que el esquema de test se desincronice del esquema real del proyecto.
- Asumir que un test que pasa con In-Memory Provider garantiza que el mismo codigo funcionara correctamente contra SQL Server real.
- No considerar el tiempo adicional de Testcontainers en el diseño de la suite de tests, generando pipelines de CI excesivamente lentos sin optimizacion.

## Practica

- [ ] Escribe un test usando el In-Memory Provider y identifica un caso donde su comportamiento difiere de SQL Server real.
- [ ] Configura Testcontainers con la imagen oficial de SQL Server para un test de integracion.
- [ ] Aplica Migrations reales dentro del setup de un test con Testcontainers.
- [ ] Compara el tiempo de ejecucion entre In-Memory Provider y Testcontainers.
- [ ] Define tu propio criterio de cuando usarias cada estrategia de testing en un proyecto real.

## Preguntas de repaso

- Que limitacion critica tiene el In-Memory Provider frente a SQL Server real?
- Que ventaja ofrece Testcontainers sobre el In-Memory Provider, y a que costo?
- Por que es importante aplicar Migrations reales dentro de un test con Testcontainers?
- Como se relaciona esta estrategia con el principio de "no mockear la base de datos en integration tests"?

## Cierre del capitulo

Con este tema cierras el capitulo de SQL Server con .NET y EF Core, habiendo cubierto la conexion base, el flujo completo de Migrations, la decision entre LINQ y Stored Procedures, el debate del Repository Pattern en el contexto de Clean Architecture, y estrategias de testing fieles al comportamiento real de produccion.
