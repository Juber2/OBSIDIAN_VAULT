---
tags:
  - NET
  - Polly
Creado: 2026-05-21
Relacionado:
  - Microservicios .NET
---

# Resiliencia con Polly

Polly ayuda con retries, circuit breakers, timeouts y fallbacks.

## Patrones

| Patrón | Uso |
|---|---|
| Retry | fallos temporales |
| Circuit breaker | evitar insistir a dependencia caída |
| Timeout | limitar espera |
| Fallback | respuesta alternativa |

## Cuidado

Retries sin backoff pueden empeorar una caída.

## Checklist

- [ ] Uso retry solo para errores transitorios.
- [ ] Tengo timeout.
- [ ] Circuit breaker protege dependencias.
- [ ] No duplico operaciones no idempotentes.
