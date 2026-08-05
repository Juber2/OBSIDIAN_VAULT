---
tags:
  - CSharp
  - ExtensionMethods
Creado: 2026-05-21
Relacionado:
  - CSharp Intermedio
---

# Extension Methods

Extension methods agregan métodos a tipos existentes sin modificarlos.

```csharp
public static class StringExtensions
{
    public static bool IsBlank(this string? value)
    {
        return string.IsNullOrWhiteSpace(value);
    }
}
```

## Uso real

- Configurar servicios en ASP.NET Core.
- Helpers de dominio simples.
- Conversión de DTOs cuando no amerita mapper dedicado.

## Cuidado

Si todo es extension method, el código se vuelve difícil de descubrir.

## Checklist

- [ ] Uso extensiones para mejorar legibilidad.
- [ ] No oculto lógica de negocio compleja.
- [ ] Las agrupo en namespaces claros.
- [ ] Evito nombres demasiado genéricos.
