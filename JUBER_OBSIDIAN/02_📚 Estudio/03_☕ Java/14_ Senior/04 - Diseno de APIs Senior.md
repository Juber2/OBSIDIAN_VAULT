---
tags:
  - Java
  - API
Creado: 2026-05-21
Relacionado:
  - Senior Java Engineer
---

# Diseno de APIs Senior

Una API buena es predecible, consistente y facil de consumir.

## Principios

- Usa recursos claros: `/users`, `/orders/{id}`.
- Usa status HTTP correctamente.
- Versiona cambios incompatibles.
- Devuelve errores consistentes.
- Documenta contratos.

## Error response

```json
{
  "code": "USER_EMAIL_ALREADY_EXISTS",
  "message": "Email already exists",
  "traceId": "abc-123"
}
```

## Paginacion

```http
GET /orders?page=0&size=20&sort=createdAt,desc
```

## Checklist

- [ ] Los endpoints tienen nombres consistentes.
- [ ] Los errores son estables para frontend/integradores.
- [ ] No expongo campos internos.
- [ ] Uso OpenAPI/Swagger cuando aplica.
- [ ] Tengo compatibilidad hacia atras.
