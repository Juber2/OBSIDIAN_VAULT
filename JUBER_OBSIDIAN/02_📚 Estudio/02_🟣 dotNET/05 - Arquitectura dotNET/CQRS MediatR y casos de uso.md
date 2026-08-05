---
tags:
  - dotnet
  - Avanzado
  - arquitectura
Creado: 2026-05-29
Nivel: Avanzado
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Arquitectura dotNET
  - NET
---

# CQRS MediatR y casos de uso

## CQRS

Separar comandos que cambian estado de queries que leen. No exige bases separadas.

## MediatR

Puede ordenar handlers, pipelines y cross-cutting concerns, pero no debe ocultar diseño pobre.

## Comando ejemplo


```csharp
public sealed record CreateUserCommand(string Email, string Name) : IRequest<Guid>;

public sealed class CreateUserHandler(AppDbContext db)
    : IRequestHandler<CreateUserCommand, Guid>
{
    public async Task<Guid> Handle(CreateUserCommand command, CancellationToken ct)
    {
        var user = User.Create(command.Email, command.Name);
        db.Users.Add(user);
        await db.SaveChangesAsync(ct);
        return user.Id;
    }
}
```


## Criterio

Usa CQRS cuando lecturas y escrituras tienen necesidades distintas, o cuando quieres casos de uso explícitos. Para CRUD pequeño, endpoints + services pueden bastar.

## Errores comunes

- Handlers que solo llaman repository sin aportar
- Commands con entidades EF completas
- Pipelines mágicos sin documentar

## Practica

- [ ] Crear command/query para Users

## Relacionado

- [[Arquitectura dotNET]]
- [[NET - Roadmap]]
