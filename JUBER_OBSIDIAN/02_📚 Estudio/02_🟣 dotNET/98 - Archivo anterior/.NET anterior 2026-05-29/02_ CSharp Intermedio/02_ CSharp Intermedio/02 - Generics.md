---
tags:
  - CSharp
  - Generics
Creado: 2026-05-21
Relacionado:
  - CSharp Intermedio
---

# Generics

Generics permiten escribir código reutilizable con tipos seguros.

```csharp
public class Result<T>
{
    public bool IsSuccess { get; init; }
    public T? Value { get; init; }
    public string? Error { get; init; }
}
```

## Constraints

```csharp
public T Create<T>() where T : new()
{
    return new T();
}
```

## Uso real

- Repositorios.
- Result pattern.
- Validadores.
- Servicios base.
- Wrappers de respuesta.

## Checklist

- [ ] Uso generics para evitar duplicación real.
- [ ] Agrego constraints cuando hacen explícito el contrato.
- [ ] No creo abstracciones genéricas prematuras.
- [ ] Mantengo nombres claros: `T`, `TEntity`, `TResult`.
