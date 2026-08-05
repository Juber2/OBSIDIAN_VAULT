---
tags:
  - dotnet
  - Basico
  - csharp
Creado: 2026-05-29
Nivel: Basico
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - CSharp y Runtime dotNET
  - NET
---

# Async Await Task CancellationToken

## Task

Representa una operacion asincrona. En APIs web libera threads mientras espera IO: base de datos, HTTP, archivos.

## CancellationToken

Permite cortar trabajo cuando el cliente cancela request o vence timeout. Debe viajar desde controller hasta EF/HTTP client.

## Ejemplo API


```csharp
[HttpGet("{id:guid}")]
public async Task<ActionResult<UserResponse>> Get(Guid id, CancellationToken ct)
{
    var user = await usersService.GetByIdAsync(id, ct);
    return user is null ? NotFound() : Ok(user);
}
```


## Regla practica

Async no acelera CPU. Mejora escalabilidad en IO. Para CPU-bound usa otras estrategias: background services, colas o paralelismo controlado.

## Errores comunes

- async void
- Bloquear con .Result o .Wait()
- No pasar CancellationToken

## Practica

- [ ] Propagar CancellationToken en un endpoint completo

## Relacionado

- [[CSharp y Runtime dotNET]]
- [[NET - Roadmap]]
