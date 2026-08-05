---
tags:
  - CSharp
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - .NET Fundamentos
---

# Métodos, Parámetros y Overloads

Los métodos encapsulan comportamiento y ayudan a expresar intención.

```csharp
decimal CalculateTotal(decimal price, int quantity)
{
    if (quantity <= 0) throw new ArgumentException("Quantity must be positive");
    return price * quantity;
}
```

## Overload

```csharp
User FindUser(int id) => repository.Find(id);
User FindUser(string email) => repository.FindByEmail(email);
```

## Parámetros opcionales

```csharp
void Log(string message, string level = "Info")
{
    Console.WriteLine($"[{level}] {message}");
}
```

## Checklist

- [ ] Cada método tiene una responsabilidad clara.
- [ ] Evito demasiados parámetros.
- [ ] Uso nombres que expresan intención.
- [ ] Valido precondiciones importantes.
