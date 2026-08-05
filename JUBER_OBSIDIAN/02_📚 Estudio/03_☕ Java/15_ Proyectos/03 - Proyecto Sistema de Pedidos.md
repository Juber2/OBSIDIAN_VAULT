---
tags:
  - Java
  - JPA
  - Proyecto
Creado: 2026-05-21
Relacionado:
  - Proyectos Java
---

# Proyecto Sistema de Pedidos

## Objetivo

Modelar usuarios, productos, pedidos y pagos con reglas reales de negocio.

## Requisitos

- Crear pedido con lineas.
- Calcular total.
- Cambiar estado del pedido.
- Persistir con JPA.
- Migraciones con Flyway.
- Evitar N+1.

## Modelo base

- `User`
- `Product`
- `Order`
- `OrderLine`
- `Payment`

## Checklist

- [ ] El dominio protege invariantes.
- [ ] Las transacciones estan en casos de uso.
- [ ] Hay migraciones.
- [ ] Hay tests de integracion con TestContainers.
