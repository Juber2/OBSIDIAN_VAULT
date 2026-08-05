---
tags:
  - NET
  - ASPNETCore
  - Performance
Creado: 2026-05-21
Relacionado:
  - Performance .NET
---

# Performance ASP.NET Core

La latencia en APIs suele venir de base de datos, red, serialización o dependencias externas.

## Reglas

- Usa async para IO.
- Define timeouts.
- Pagina resultados.
- Evita payloads enormes.
- Usa `AsNoTracking` en lecturas EF.

## Checklist

- [ ] No bloqueo threads con `.Result`.
- [ ] Uso cancellation tokens.
- [ ] Mido queries por request.
- [ ] Tengo logs y métricas.
