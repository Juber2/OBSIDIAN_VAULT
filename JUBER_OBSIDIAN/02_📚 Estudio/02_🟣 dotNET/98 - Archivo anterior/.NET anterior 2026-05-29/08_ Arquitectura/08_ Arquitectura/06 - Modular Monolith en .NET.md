---
tags:
  - NET
  - ModularMonolith
Creado: 2026-05-21
Relacionado:
  - Arquitectura .NET
---

# Modular Monolith en .NET

Un monolito modular se despliega como una app, pero internamente respeta módulos de negocio.

```text
src/
  Modules/
    Sales/
    Billing/
    Inventory/
  SharedKernel/
```

## Reglas

- Un módulo no accede a tablas internas de otro.
- Los contratos públicos son explícitos.
- Eventos internos comunican cambios relevantes.
- `SharedKernel` debe ser pequeño.

## Checklist

- [ ] Los módulos representan capacidades.
- [ ] Evito referencias circulares.
- [ ] Puedo extraer un módulo si hace falta.
- [ ] Mantengo boundaries claros.
