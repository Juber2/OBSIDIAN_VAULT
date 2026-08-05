---
tags:
  - ASPNETCore
  - Errores
Creado: 2026-05-21
Relacionado:
  - ASP.NET Core
---

# Manejo Global de Errores

Una API senior devuelve errores consistentes.

## ProblemDetails

```json
{
  "type": "https://httpstatuses.com/404",
  "title": "Resource not found",
  "status": 404,
  "traceId": "abc-123"
}
```

## Reglas

- 400: error de request.
- 401: no autenticado.
- 403: sin permiso.
- 404: recurso no encontrado.
- 409: conflicto de negocio.
- 500: error inesperado.

## Checklist

- [ ] Tengo middleware/handler global.
- [ ] No expongo stack traces al cliente.
- [ ] Logueo errores inesperados.
- [ ] Uso códigos de error estables.
