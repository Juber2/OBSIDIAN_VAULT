---
tags:
  - dotnet
  - Avanzado
  - testing
Creado: 2026-05-29
Nivel: Avanzado
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Testing Calidad dotNET
  - NET
---

# Integration Tests WebApplicationFactory

## Integration test

Levanta la API casi real y prueba pipeline, DI, routing, auth, serialization y DB test.

## Ejemplo


```csharp
public sealed class UsersTests : IClassFixture<WebApplicationFactory<Program>>
{
    [Fact]
    public async Task Get_users_returns_ok()
    {
        var client = factory.CreateClient();
        var response = await client.GetAsync("/api/v1/users");
        response.StatusCode.Should().Be(HttpStatusCode.OK);
    }
}
```


## Valor

Detecta problemas que unit tests no ven: middleware faltante, DI roto, rutas mal escritas, JSON incorrecto.

## Errores comunes

- Usar solo unit tests para API
- Compartir estado entre tests
- No resetear DB

## Practica

- [ ] Crear test de login y endpoint protegido

## Relacionado

- [[Testing Calidad dotNET]]
- [[NET - Roadmap]]
