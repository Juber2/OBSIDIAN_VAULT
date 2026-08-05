---
tags:
  - CSharp
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - .NET Fundamentos
---

# Control de Flujo

Control de flujo define qué caminos toma el programa.

## Condicionales

```csharp
if (age >= 18)
{
    Console.WriteLine("Mayor de edad");
}
else
{
    Console.WriteLine("Menor de edad");
}
```

## Switch expression

```csharp
string label = status switch
{
    "P" => "Pendiente",
    "A" => "Aprobado",
    "R" => "Rechazado",
    _ => "Desconocido"
};
```

## Loops

```csharp
foreach (var item in items)
{
    Console.WriteLine(item);
}
```

## Checklist

- [ ] Uso `switch expression` cuando mejora legibilidad.
- [ ] Evito condicionales enormes.
- [ ] Uso `foreach` para recorrer colecciones.
- [ ] Sé cuándo cortar con `break` o `return`.
