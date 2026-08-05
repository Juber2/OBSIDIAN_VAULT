---
tags:
  - ASPNETCore
  - API
Creado: 2026-05-21
Relacionado:
  - ASP.NET Core
---

# Controllers, Minimal APIs y Routing

ASP.NET Core permite crear APIs con controllers o Minimal APIs.

## Controller

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id:int}")]
    public async Task<ActionResult<ProductDto>> Get(int id)
    {
        return Ok(await service.GetAsync(id));
    }
}
```

## Minimal API

```csharp
app.MapGet("/api/products/{id:int}", async (int id, IProductService service) =>
{
    var product = await service.GetAsync(id);
    return Results.Ok(product);
});
```

## Criterio

Controllers suelen ir bien en APIs grandes. Minimal APIs brillan en servicios pequeños, endpoints simples o APIs muy funcionales.

## Checklist

- [ ] Uso rutas consistentes.
- [ ] Valido status codes.
- [ ] No retorno entidades internas.
- [ ] Elijo controller/minimal según contexto.
