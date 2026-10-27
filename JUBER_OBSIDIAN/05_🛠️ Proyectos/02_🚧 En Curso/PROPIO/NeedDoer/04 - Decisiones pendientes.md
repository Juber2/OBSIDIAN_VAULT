---
tipo: registro de decisiones
estado: abierto
actualizado: 2026-10-17
tags:
  - needdoer
  - decisiones
---

# Decisiones pendientes

Volver a [[NeedDoer]].

## Decisiones explícitamente abiertas en el mapa inicial

### DP-001 — Momento de apertura del chat

- **Estado:** resuelta.
- **Decisión:** el chat se habilita solo después de aceptar una cotización y mientras el servicio esté activo.
- **Motivo:** permitir la coordinación directa únicamente cuando ya existe un servicio acordado.
- **Fecha y responsable:** 2026-10-18 · Juber.

### DP-002 — Antecedentes del prestador

- **Estado:** pendiente.
- **Opciones:** exigir CertiAdulto o CertiJoven; hacerlo opcional para obtener un sello adicional; dejarlo fuera del MVP.
- **Impacto:** confianza, fricción de alta, revisión manual y tratamiento de datos sensibles.
- **Decisión:**
- **Motivo:**
- **Fecha y responsable:**

## Propuestas que deben validarse

- [ ] DP-003 — Valor del crédito y costo de cada cotización aceptada.
- [ ] DP-004 — Máximo de cinco cotizaciones por solicitud.
- [ ] DP-005 — Vigencia de solicitudes urgentes y programadas.
- [ ] DP-006 — Casos de devolución después de que una cotización ya fue aceptada y cobrada.
- [ ] DP-007 — Diez créditos de bienvenida.
- [ ] DP-008 — Finalización automática después de 48 horas.
- [ ] DP-009 — Umbrales de cancelación y revisión de cuentas.
- [ ] DP-010 — Umbral de calificación baja.

Los valores actuales y sus motivos están en [[03 - Reglas de negocio]].

## Definiciones necesarias antes del desarrollo

Estas preguntas se desprenden del alcance, pero no están resueltas en el archivo base:

- [ ] DP-011 — Distritos y ciudad del lanzamiento inicial.
- [ ] DP-012 — Proveedor de SMS para registro e inicio de sesión.
- [ ] DP-013 — Proveedor de pagos para Yape y tarjetas al comprar créditos.
- [ ] DP-014 — Proveedor de mapas, geocodificación y cálculo de distancia.
- [ ] DP-015 — Plazo de conservación y proceso de eliminación de DNI, selfie, chats y comprobantes.
- [ ] DP-016 — Tiempo de respuesta y canales del soporte.
- [ ] DP-017 — Política de reembolsos de compras de créditos.
- [ ] DP-018 — Número de fotos, peso máximo y formatos admitidos después de la compresión.
- [ ] DP-019 — Tratamiento de solicitudes sin cotizaciones y alcance al ampliar la zona.
- [ ] DP-020 — Qué ocurre en una disputa por monto distinto, ausencia o trabajo deficiente.
- [ ] DP-021 — Si el crédito solo se valida o también se reserva mientras la cotización está vigente, y qué ocurre si el prestador tiene varias cotizaciones abiertas con poco saldo.

## Plantilla para nuevas decisiones

```markdown
### DP-XXX — Título

- **Estado:** pendiente | aprobada | descartada.
- **Contexto:**
- **Opciones:**
- **Decisión:**
- **Motivo:**
- **Impacto en funcionalidades o reglas:**
- **Fecha y responsable:**
```
