---
tags:
  - NET
  - xUnit
Creado: 2026-05-21
Relacionado:
  - Testing en .NET
---

# xUnit Fundamentos

xUnit es un framework de testing muy usado en .NET.

```csharp
public class CalculatorTests
{
    [Fact]
    public void Add_ShouldReturnSum()
    {
        var result = Calculator.Add(2, 3);
        Assert.Equal(5, result);
    }
}
```

## Theory

```csharp
[Theory]
[InlineData(2, 3, 5)]
[InlineData(10, 5, 15)]
public void Add_ShouldReturnExpected(int a, int b, int expected)
{
    Assert.Equal(expected, Calculator.Add(a, b));
}
```

## Checklist

- [ ] Uso nombres descriptivos.
- [ ] Testeo un comportamiento por test.
- [ ] Uso `Theory` para casos repetidos.
- [ ] Evito tests dependientes del orden.
