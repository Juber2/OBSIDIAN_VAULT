---
tags:
  - Java
  - Spring Boot
  - Performance
Creado: 2026-05-21
Relacionado:
  - Performance Java
---

# Performance en Spring Boot

La performance en Spring suele depender mas de base de datos, serializacion, transacciones y llamadas externas que del framework en si.

## Puntos criticos

| Area | Riesgo |
|---|---|
| JPA | N+1, lazy loading accidental |
| HTTP | payloads grandes, timeouts ausentes |
| JSON | serializacion costosa |
| Transacciones | demasiado largas |
| Pools | conexiones insuficientes |

## Base de datos

Antes de tocar JVM, revisa:

- indices,
- queries lentas,
- numero de queries por request,
- tamano de pagina,
- transacciones abiertas.

## Checklist

- [ ] No retorno entidades JPA completas.
- [ ] Uso paginacion.
- [ ] Tengo timeouts.
- [ ] Mido queries por endpoint.
- [ ] Uso cache solo donde hay patron de lectura repetido.
