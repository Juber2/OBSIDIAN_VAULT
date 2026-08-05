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

# 02 - EF Core Migrations

## Que vas a aprender

En este capitulo vas a aprender **Migrations** de Entity Framework Core: el enfoque **Code First** (definir clases C# y generar el esquema a partir de ellas) frente a **Database First** (generar clases C# a partir de un esquema existente), y el flujo completo de `Add-Migration`/`Update-Database`, aplicando directamente el contexto ya conocido de StaffTrainingPlatform.

## Explicacion

Con **Code First** (el enfoque predominante en proyectos nuevos, incluyendo StaffTrainingPlatform), se definen clases C# (entidades) y un `DbContext`, y EF Core genera automaticamente el SQL necesario para crear/modificar el esquema mediante **Migrations**: cada `dotnet ef migrations add NombreMigracion` genera un archivo C# con metodos `Up()`/`Down()` describiendo el cambio incremental, similar en espiritu a las migraciones de TypeORM/Prisma ya estudiadas en el libro de NestJS, aunque expresadas en C# en vez de TypeScript/SQL.

**Database First** invierte el flujo: se parte de una base de datos ya existente (comun en proyectos legacy o cuando el DBA controla el esquema directamente) y se genera el modelo de C# a partir de ella con `Scaffold-DbContext`, util para integrarse con sistemas donde el esquema no esta bajo control del codigo de la aplicacion. La mayoria de proyectos nuevos, incluido StaffTrainingPlatform, usan Code First porque mantiene el esquema versionado junto con el codigo, permitiendo revision en Pull Requests igual que cualquier otro cambio.

## Modelo mental

Pensa Code First como diseñar primero los planos de una casa (las clases C#) y dejar que un contratista (EF Core) construya la casa fisica (el esquema de base de datos) a partir de esos planos, actualizandola cada vez que modificas el plano (una nueva Migration). Database First es lo inverso: la casa ya existe, y contratas a alguien para que dibuje los planos a partir de lo que ya esta construido.

## Ejemplo base

```csharp
// Entidad C# (Code First)
public class Curso
{
    public int Id { get; set; }
    public string Nombre { get; set; } = string.Empty;
    public int DuracionHoras { get; set; }
}

public class AppDbContext : DbContext
{
    public DbSet<Curso> Cursos => Set<Curso>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlServer("Server=localhost;Database=MiApp;...");
}
```

```bash
# Flujo de Migrations
dotnet ef migrations add AgregarTablaCursos
dotnet ef database update

# Database First: generar clases desde un esquema existente
dotnet ef dbcontext scaffold "Server=...;Database=...;" Microsoft.EntityFrameworkCore.SqlServer
```

## Como se usa en un proyecto real

1. Se usa Code First en StaffTrainingPlatform, definiendo entidades C# y generando Migrations versionadas junto con el codigo en el repositorio.
2. Se revisa cada Migration generada antes de commitear, verificando que el `Up()`/`Down()` generado refleje realmente el cambio intencionado.
3. Se aplica el flujo de migraciones seguras hacia la nube ya visto en el capitulo anterior (script idempotente via pipeline), no `database update` directo contra produccion.
4. Se conecta esto con las migraciones de TypeORM/Prisma del libro de NestJS, reconociendo el mismo problema resuelto con herramientas distintas.
5. Se usa Database First unicamente en escenarios de integracion con bases de datos legacy fuera del control del equipo de desarrollo.

## Criterio junior

Un junior debe poder definir una entidad C#, generar una Migration, y aplicarla con `database update` en desarrollo local.

## Criterio mid

Un mid debe revisar criticamente el contenido de una Migration generada automaticamente antes de aplicarla.

## Criterio senior

Un senior debe decidir entre Code First y Database First segun el contexto del proyecto, y garantizar que el flujo de Migrations este integrado de forma segura con CI/CD.

## Errores comunes

- No revisar el contenido de una Migration generada automaticamente, aplicando cambios no intencionados al esquema.
- Editar manualmente una Migration ya aplicada en otros entornos, causando inconsistencia entre el historial de migraciones y el estado real de cada base de datos.
- Usar Database First en un proyecto nuevo donde Code First seria mas apropiado, perdiendo el versionado del esquema junto al codigo.
- No aplicar Migrations en el mismo orden en todos los entornos, causando el mismo tipo de problemas de esquema desincronizado que ya se vio con TypeORM/Prisma en NestJS.

## Practica

- [ ] Define una entidad C# y genera su primera Migration con `dotnet ef migrations add`.
- [ ] Aplica la Migration con `dotnet ef database update` y verifica el esquema resultante.
- [ ] Modifica la entidad y genera una segunda Migration incremental.
- [ ] Investiga el proceso de `Scaffold-DbContext` para Database First.
- [ ] Compara este flujo con las migraciones de TypeORM/Prisma ya estudiadas en el libro de NestJS.

## Preguntas de repaso

- Que diferencia hay entre Code First y Database First?
- Por que es importante revisar el contenido de una Migration antes de aplicarla?
- En que escenario elegirias Database First sobre Code First?
- Como se compara el flujo de Migrations de EF Core con el de TypeORM/Prisma?
