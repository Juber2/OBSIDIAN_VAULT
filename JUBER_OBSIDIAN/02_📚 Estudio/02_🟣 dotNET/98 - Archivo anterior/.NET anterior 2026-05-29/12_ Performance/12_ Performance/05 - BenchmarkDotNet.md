---
tags:
  - NET
  - Benchmark
Creado: 2026-05-21
Relacionado:
  - Performance .NET
---

# BenchmarkDotNet

BenchmarkDotNet es la herramienta estándar para microbenchmarks en .NET.

```csharp
[MemoryDiagnoser]
public class StringBenchmarks
{
    [Benchmark]
    public string Concat() => string.Join(",", Enumerable.Range(1, 100));
}
```

## Checklist

- [ ] Uso BenchmarkDotNet, no cronómetro manual.
- [ ] Reviso tiempo y memoria.
- [ ] No confundo microbenchmark con carga real.
- [ ] Comparo alternativas bajo condiciones similares.
