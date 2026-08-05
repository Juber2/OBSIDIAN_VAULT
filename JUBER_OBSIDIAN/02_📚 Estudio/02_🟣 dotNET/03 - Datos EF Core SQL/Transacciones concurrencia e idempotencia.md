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

# Transacciones concurrencia e idempotencia

## Transacciones

Agrupan cambios que deben confirmarse juntos. En EF, SaveChanges ya usa transaccion para varios cambios en una llamada.

## Concurrencia optimista

Usa rowversion/concurrency token cuando dos usuarios pueden editar lo mismo.

## Idempotencia

En endpoints críticos evita duplicar operaciones cuando el cliente reintenta. Útil en pagos, inscripciones y procesamiento de mensajes.

## Ejemplo rowversion


```csharp
modelBuilder.Entity<Course>()
    .Property<byte[]>("RowVersion")
    .IsRowVersion();
```


## Errores comunes

- Transacciones demasiado largas
- No manejar DbUpdateConcurrencyException
- Endpoints POST que duplican datos en retry

## Practica

- [ ] Diseñar update con control de concurrencia

## Relacionado

- [[Datos EF Core SQL]]
- [[NET - Roadmap]]
