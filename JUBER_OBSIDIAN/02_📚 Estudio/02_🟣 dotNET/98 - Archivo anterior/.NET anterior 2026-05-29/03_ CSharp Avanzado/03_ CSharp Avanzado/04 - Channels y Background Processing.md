---
tags:
  - NET
  - BackgroundService
Creado: 2026-05-21
Relacionado:
  - CSharp Avanzado
---

# Channels y Background Processing

Channels permiten colas en memoria para productores y consumidores async.

```csharp
var channel = Channel.CreateUnbounded<string>();
await channel.Writer.WriteAsync("job");
var item = await channel.Reader.ReadAsync();
```

## BackgroundService

```csharp
public class Worker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

## Usos

- Procesar trabajos internos.
- Enviar emails.
- Desacoplar requests de tareas lentas.

## Checklist

- [ ] Respeto `CancellationToken`.
- [ ] Manejo errores dentro del worker.
- [ ] No pierdo trabajos críticos si reinicia la app.
- [ ] Uso cola externa si necesito durabilidad.
