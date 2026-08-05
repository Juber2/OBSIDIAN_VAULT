---
tags:
  - CSharp
  - Performance
Creado: 2026-05-21
Relacionado:
  - CSharp Avanzado
---

# Span, Memory y Performance

`Span<T>` permite trabajar con segmentos de memoria sin copiar datos.

```csharp
ReadOnlySpan<char> text = "ABC-123";
var prefix = text[..3];
```

## Cuándo importa

- Parsers.
- Procesamiento de texto grande.
- Protocolos.
- Serialización.
- Código de alta performance.

## Diferencia

| Tipo | Uso |
|---|---|
| `Span<T>` | stack-only, síncrono |
| `ReadOnlySpan<T>` | lectura sin copia |
| `Memory<T>` | puede vivir en heap y cruzar async |

## Checklist

- [ ] No uso `Span<T>` por moda.
- [ ] Lo aplico donde evita copias reales.
- [ ] Entiendo que `Span<T>` no cruza `await`.
- [ ] Mido antes y después.
