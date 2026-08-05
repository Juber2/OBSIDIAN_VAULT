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

# Mocking boundaries y servicios externos

## Qué mockear

Mockea puertos externos: email, storage, HTTP externo, clock, current user. Evita mockear modelos ricos o LINQ interno.

## Ejemplo con interfaz


```csharp
emailSender
    .SendAsync(Arg.Any<EmailMessage>(), Arg.Any<CancellationToken>())
    .Returns(Task.CompletedTask);
```


## Boundary

Si necesitas muchos mocks para un caso simple, quizá el caso de uso tiene demasiadas responsabilidades.

## Errores comunes

- Mockear DbSet complejo
- Tests llenos de setup irrelevante
- Verificar llamadas internas en vez de resultado

## Practica

- [ ] Extraer IClock y testear expiracion

## Relacionado

- [[Testing Calidad dotNET]]
- [[NET - Roadmap]]
