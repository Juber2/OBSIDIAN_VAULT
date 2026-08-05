---
tags:
  - EFCore
  - Transacciones
Creado: 2026-05-21
Relacionado:
  - Entity Framework Core
---

# Transacciones, Unit of Work y Concurrencia

`DbContext` funciona como Unit of Work: agrupa cambios y los guarda con `SaveChanges`.

```csharp
await using var transaction = await db.Database.BeginTransactionAsync(ct);

// cambios
await db.SaveChangesAsync(ct);
await transaction.CommitAsync(ct);
```

## Concurrencia optimista

```csharp
public byte[] RowVersion { get; set; } = [];
```

Configura row version para detectar actualizaciones simultáneas.

## Checklist

- [ ] Sé dónde empieza y termina una transacción.
- [ ] Mantengo transacciones cortas.
- [ ] Manejo conflictos de concurrencia.
- [ ] No mezclo operaciones externas dentro de transacción DB si puedo evitarlo.
