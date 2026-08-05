---
tags:
  - CSharp
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - .NET Fundamentos
---

# Sintaxis, Variables y Tipos

C# es estáticamente tipado. El compilador conoce los tipos antes de ejecutar.

## Tipos comunes

| Tipo | Uso |
|---|---|
| `int`, `long` | números enteros |
| `decimal` | dinero y precisión decimal |
| `double` | cálculos científicos/aproximados |
| `bool` | verdadero/falso |
| `string` | texto |
| `DateTime`, `DateOnly` | fechas |
| `Guid` | identificadores |

```csharp
string name = "Juber";
int age = 25;
decimal price = 99.90m;
bool active = true;
```

## `var`

`var` no es dinámico. El compilador infiere el tipo.

```csharp
var total = 10m; // decimal
```

## Checklist

- [ ] Uso `decimal` para dinero.
- [ ] Entiendo inferencia con `var`.
- [ ] Diferencio tipos valor y referencia.
- [ ] Evito conversiones implícitas peligrosas.
