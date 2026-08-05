---
tags:
  - CSharp
  - Concurrencia
Creado: 2026-05-21
Relacionado:
  - CSharp Avanzado
---

# Threading, Task Parallel y Locks

Concurrencia permite hacer varias cosas al mismo tiempo, pero introduce riesgos de estado compartido.

```csharp
private readonly object _lock = new();

lock (_lock)
{
    counter++;
}
```

## Herramientas

| Herramienta | Uso |
|---|---|
| `Task` | operación asíncrona |
| `Parallel.ForEach` | CPU-bound paralelo |
| `lock` | proteger sección crítica |
| `SemaphoreSlim` | limitar concurrencia |
| `ConcurrentDictionary` | colección thread-safe |

## Checklist

- [ ] Diferencio IO-bound y CPU-bound.
- [ ] Evito estado compartido mutable.
- [ ] Uso `SemaphoreSlim` para límites async.
- [ ] Reviso deadlocks potenciales.
