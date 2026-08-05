---
tags:
  - CSharp
  - POO
Creado: 2026-05-21
Relacionado:
  - .NET Fundamentos
---

# POO, Clases, Objetos y Encapsulación

POO ayuda a modelar comportamiento y datos relacionados.

```csharp
public class BankAccount
{
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount)
    {
        if (amount <= 0) throw new ArgumentException("Amount must be positive");
        Balance += amount;
    }
}
```

## Encapsulación

No expongas setters si rompen reglas del negocio. El objeto debe proteger sus invariantes.

## Checklist

- [ ] Las propiedades no son públicas por comodidad.
- [ ] Los métodos expresan acciones del dominio.
- [ ] Las reglas viven cerca de los datos que protegen.
- [ ] Evito clases con solo getters/setters cuando hay lógica real.
