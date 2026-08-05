---
tags:
  - CSharp
  - Errores
Creado: 2026-05-21
Relacionado:
  - .NET Fundamentos
---

# Excepciones y Manejo de Errores

Las excepciones representan errores inesperados o flujos que no pueden continuar.

```csharp
try
{
    var user = service.GetUser(id);
}
catch (UserNotFoundException ex)
{
    logger.LogWarning(ex, "User not found {UserId}", id);
}
```

## Buenas prácticas

- No captures `Exception` si no sabes qué hacer.
- No ocultes errores.
- Crea excepciones de dominio cuando aporten claridad.
- En APIs, traduce excepciones a respuestas HTTP consistentes.

## Checklist

- [ ] Capturo excepciones específicas.
- [ ] Logueo con contexto.
- [ ] No retorno `null` para errores importantes.
- [ ] No uso excepciones para flujo normal masivo.
