---
tags:
  - Java
  - Arquitectura
Creado: 2026-05-21
Relacionado:
  - Arquitectura Java
---

# ADR y Tradeoffs

Un ADR (Architecture Decision Record) documenta una decision importante: contexto, opciones, decision y consecuencias.

## Plantilla simple

```md
# ADR-001 - Usar PostgreSQL como base principal

## Contexto
Necesitamos persistencia relacional, transacciones y reportes.

## Opciones
- PostgreSQL
- MySQL
- MongoDB

## Decision
Usaremos PostgreSQL.

## Consecuencias
- Buen soporte transaccional.
- El equipo debe dominar indices y migraciones.
```

## Tradeoffs comunes en Java

| Decision | Ganas | Pagas |
|---|---|---|
| Monolito modular | Simplicidad operativa | Disciplina modular |
| Microservicios | Escalado independiente | Complejidad distribuida |
| JPA | Productividad | Riesgo de queries ocultas |
| Kafka | Desacoplamiento | Operacion mas dificil |
| Clean Architecture | Testabilidad | Mas clases |

## Mentalidad senior

No existe arquitectura perfecta. Existe arquitectura adecuada para el contexto, equipo, riesgo y momento del producto.

## Checklist

- [ ] Documento decisiones que seran costosas de revertir.
- [ ] Explico alternativas rechazadas.
- [ ] Incluyo consecuencias negativas.
- [ ] Reviso ADRs cuando el contexto cambia.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

