---
tags:
  - dotnet
  - Intermedio
  - efcore
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Datos EF Core SQL
  - NET
---

# DbContext entidades y configuracion

## DbContext

Unidad de trabajo y gateway hacia base de datos. No debería vivir más que un request en una API típica.

## Configuracion por Fluent API

Mantiene reglas de persistencia fuera de entidades si el modelo crece.

## Ejemplo


```csharp
public sealed class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public DbSet<User> Users => Set<User>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>(b =>
        {
            b.HasKey(x => x.Id);
            b.Property(x => x.Email).HasMaxLength(320).IsRequired();
            b.HasIndex(x => x.Email).IsUnique();
        });
    }
}
```


## Errores comunes

- DbContext como singleton
- Entidades anemicas por miedo a EF
- No configurar indices unicos

## Practica

- [ ] Modelar User con Email unico

## Relacionado

- [[Datos EF Core SQL]]
- [[NET - Roadmap]]
