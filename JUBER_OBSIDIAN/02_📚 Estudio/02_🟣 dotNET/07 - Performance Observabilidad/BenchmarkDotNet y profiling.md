---
tags:
  - dotnet
  - Senior
  - performance
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Performance Observabilidad dotNET
  - NET
---

# BenchmarkDotNet y profiling

## Benchmark

Sirve para medir codigo pequeño de forma controlada. No reemplaza medicion en produccion.

## Ejemplo


```csharp
[MemoryDiagnoser]
public class MapperBenchmarks
{
    [Benchmark]
    public UserDto Map() => mapper.Map(user);
}
```


## Profiling real

Para endpoint lento mira DB, red, serializacion, locks, CPU y allocations. El bottleneck rara vez está donde uno cree.

## Errores comunes

- Benchmark sin baseline
- Conclusiones por una sola corrida
- Optimizar mapper cuando la DB tarda 900ms

## Practica

- [ ] Benchmarkear dos estrategias de mapping

## Relacionado

- [[Performance Observabilidad dotNET]]
- [[NET - Roadmap]]
