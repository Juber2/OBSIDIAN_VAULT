---
tags:
  - EFCore
  - Persistencia
Creado: 2026-05-21
Relacionado:
  - Entity Framework Core
---

# DbContext, Entidades y Configuración

`DbContext` representa una sesión con la base de datos.

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Product> Products => Set<Product>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>()
            .Property(x => x.Name)
            .HasMaxLength(120)
            .IsRequired();
    }
}
```

## Buenas prácticas

- Configura restricciones explícitas.
- No dependas solo de convenciones.
- Evita lógica de negocio en entidades EF si son entidades puramente de persistencia.

## Checklist

- [ ] DbContext tiene responsabilidad clara.
- [ ] Entidades tienen keys y constraints.
- [ ] Configuro longitudes y requeridos.
- [ ] No expongo DbContext fuera de infraestructura/casos definidos.
