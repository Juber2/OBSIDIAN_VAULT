---
tags:
  - dotnet
  - Intermedio
  - aspnetcore
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - ASP.NET Core APIs
  - NET
---

# Controllers Minimal APIs Routing

## Controllers

Buenos para APIs medianas/grandes, versionado, filtros, convenciones y separación clara por recurso.

## Minimal APIs

Buenos para endpoints simples, microservicios pequeños, health checks o APIs muy funcionales.

## Controller ejemplo


```csharp
[ApiController]
[Route("api/v1/tenant/users")]
public sealed class TenantUsersController : ControllerBase
{
    [HttpGet("{userId:guid}")]
    public async Task<ActionResult<TenantUserResponse>> Get(Guid userId, CancellationToken ct)
        => Ok(await service.GetAsync(userId, ct));
}
```


## Routing criterio

La ruta identifica recurso. El body no debe traer flags magicos para cambiar contexto. Query params son filtros, orden y paginacion.

## Errores comunes

- Endpoints que exponen verbos innecesarios
- IDs magicos en body
- No versionar API cuando hay clientes reales

## Practica

- [ ] Diseñar endpoints para Users y Tenants

## Relacionado

- [[ASP.NET Core APIs]]
- [[NET - Roadmap]]
