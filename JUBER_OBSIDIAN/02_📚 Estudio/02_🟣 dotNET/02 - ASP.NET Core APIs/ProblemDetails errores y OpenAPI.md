---
tags:
  - dotnet
  - Intermedio
  - aspnetcore
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - ASP.NET Core APIs
  - NET
---

# ProblemDetails errores y OpenAPI

## ProblemDetails

Formato estándar para errores HTTP. Evita respuestas improvisadas distintas por endpoint.

## Errores por capa

Controller traduce resultado a HTTP. Application devuelve resultados de caso de uso. Domain protege invariantes. Infrastructure captura detalles externos.

## Ejemplo respuesta


```json
{
  "type": "https://httpstatuses.com/409",
  "title": "Conflict",
  "status": 409,
  "detail": "Email already exists"
}
```


## OpenAPI

Documenta contratos, pero no reemplaza pruebas. Útil para frontend, QA y clientes externos.

## Errores comunes

- Lanzar Exception para validaciones esperadas
- Devolver 200 con success false para todo
- Exponer stack traces

## Practica

- [ ] Definir mapa de errores 400/401/403/404/409/500

## Relacionado

- [[ASP.NET Core APIs]]
- [[NET - Roadmap]]
