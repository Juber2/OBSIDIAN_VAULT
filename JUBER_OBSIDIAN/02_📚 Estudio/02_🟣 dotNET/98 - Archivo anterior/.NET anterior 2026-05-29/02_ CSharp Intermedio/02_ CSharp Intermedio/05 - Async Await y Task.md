---
tags:
  - CSharp
  - Async
Creado: 2026-05-21
Relacionado:
  - CSharp Intermedio
---

# Async, Await y Task

`async/await` permite escribir código asíncrono legible, especialmente para IO: HTTP, base de datos, archivos, colas.

```csharp
public async Task<UserDto> GetUserAsync(int id, CancellationToken ct)
{
    var user = await repository.FindAsync(id, ct);
    return mapper.ToDto(user);
}
```

## Reglas

- Usa `Task` para operaciones async sin resultado.
- Usa `Task<T>` para resultado.
- Propaga `CancellationToken`.
- No bloquees con `.Result` o `.Wait()`.

## Checklist

- [ ] Uso `Async` como sufijo.
- [ ] Propago `CancellationToken`.
- [ ] Evito bloquear código async.
- [ ] Entiendo que async no significa paralelo automáticamente.
