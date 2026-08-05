---
tags:
  - NET
  - HTTP
  - gRPC
Creado: 2026-05-21
Relacionado:
  - Microservicios .NET
---

# HttpClientFactory, Refit y gRPC

`IHttpClientFactory` gestiona clientes HTTP correctamente.

```csharp
builder.Services.AddHttpClient<IUsersClient, UsersClient>(client =>
{
    client.BaseAddress = new Uri("https://users-api");
});
```

## Opciones

| Opción | Uso |
|---|---|
| HttpClientFactory | HTTP flexible |
| Refit | cliente declarativo |
| gRPC | contratos fuertes y alto rendimiento |

## Checklist

- [ ] Uso timeouts.
- [ ] No creo `new HttpClient()` por request.
- [ ] Manejo errores remotos.
- [ ] Versiono contratos.
