---
tags:
  - NET
  - YARP
Creado: 2026-05-21
Relacionado:
  - Microservicios .NET
---

# API Gateway con YARP

YARP es un reverse proxy de Microsoft para construir gateways.

## Gateway puede manejar

- Routing.
- Autenticación central.
- Rate limiting.
- Transformaciones.
- Balanceo.

## Cuidado

No conviertas el gateway en un monolito de lógica de negocio.

## Checklist

- [ ] Gateway enruta, no contiene dominio.
- [ ] Tengo límites y auth claros.
- [ ] Versiono rutas.
- [ ] Observo latencia por servicio.
