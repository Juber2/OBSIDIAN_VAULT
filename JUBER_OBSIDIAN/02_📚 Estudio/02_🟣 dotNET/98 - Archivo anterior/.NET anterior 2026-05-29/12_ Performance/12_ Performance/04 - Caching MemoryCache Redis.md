---
tags:
  - NET
  - Cache
Creado: 2026-05-21
Relacionado:
  - Performance .NET
---

# Caching con MemoryCache y Redis

Cache evita repetir trabajo caro.

## Opciones

| Cache | Uso |
|---|---|
| `IMemoryCache` | local, simple |
| Redis | compartida, distribuida |

```csharp
var value = await cache.GetOrCreateAsync(key, entry =>
{
    entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
    return LoadAsync();
});
```

## Checklist

- [ ] Defino TTL.
- [ ] Tengo invalidación.
- [ ] No cacheo datos sensibles sin control.
- [ ] Mido hit ratio.
