---
tags:
  - NET
  - Diagnostico
Creado: 2026-05-21
Relacionado:
  - Performance .NET
---

# Diagnóstico con dotnet-counters, trace y dump

.NET trae herramientas para observar procesos.

```bash
dotnet-counters monitor --process-id 1234
dotnet-trace collect --process-id 1234
dotnet-dump collect --process-id 1234
```

## Qué mirar

- CPU.
- GC heap size.
- Gen 0/1/2 collections.
- Thread pool.
- Exceptions.
- Requests por segundo.

## Checklist

- [ ] Capturo evidencia.
- [ ] Identifico cuello de botella.
- [ ] Comparo antes/después.
- [ ] No optimizo por intuición solamente.
