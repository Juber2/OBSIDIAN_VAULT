---
tags:
  - NET
  - Testcontainers
Creado: 2026-05-21
Relacionado:
  - Testing en .NET
---

# Testcontainers con .NET

Testcontainers levanta dependencias reales en Docker para tests: PostgreSQL, SQL Server, Redis, RabbitMQ, etc.

## Uso

Sirve para probar integración con infraestructura real sin depender de ambientes compartidos.

## Beneficios

- Tests reproducibles.
- Base limpia.
- Menos mocks para persistencia.
- Mayor confianza en queries y migraciones.

## Checklist

- [ ] Uso contenedores para dependencias críticas.
- [ ] Limpio datos entre tests.
- [ ] No dependo de una DB local manual.
- [ ] Mantengo tests de integración separados de unitarios.
