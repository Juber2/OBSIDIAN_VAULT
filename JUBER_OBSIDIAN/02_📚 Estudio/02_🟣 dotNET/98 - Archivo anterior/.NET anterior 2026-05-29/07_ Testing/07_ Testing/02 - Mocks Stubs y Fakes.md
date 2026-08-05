---
tags:
  - NET
  - Testing
Creado: 2026-05-21
Relacionado:
  - Testing en .NET
---

# Mocks, Stubs y Fakes

Dobles de prueba reemplazan dependencias en tests.

## Diferencias

| Tipo | Uso |
|---|---|
| Stub | devuelve datos preparados |
| Mock | verifica interacción |
| Fake | implementación simple funcional |

```csharp
var repository = new Mock<IUserRepository>();
repository.Setup(x => x.ExistsAsync(email, ct)).ReturnsAsync(false);
```

## Regla

No mockees todo. Si el test queda más complejo que el código, revisa el diseño.

## Checklist

- [ ] Mockeo fronteras externas.
- [ ] Prefiero fakes para dominio simple.
- [ ] No verifico detalles irrelevantes.
- [ ] Los tests siguen siendo legibles.
