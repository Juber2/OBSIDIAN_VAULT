---
tags:
  - NET
  - IntegrationTesting
Creado: 2026-05-21
Relacionado:
  - Testing en .NET
---

# Integration Testing con WebApplicationFactory

`WebApplicationFactory` permite levantar la API en memoria para probar endpoints completos.

```csharp
public class ProductsApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public ProductsApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }
}
```

## Qué valida

- Routing.
- Middlewares.
- Serialización.
- DI.
- Autenticación si la configuras.

## Checklist

- [ ] Pruebo endpoints críticos.
- [ ] Uso datos aislados por test.
- [ ] Reemplazo dependencias externas.
- [ ] Verifico status code y body.
