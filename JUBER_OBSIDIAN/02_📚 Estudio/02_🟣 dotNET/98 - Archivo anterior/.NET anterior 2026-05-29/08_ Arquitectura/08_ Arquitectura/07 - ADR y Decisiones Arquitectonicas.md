---
tags:
  - NET
  - ADR
Creado: 2026-05-21
Relacionado:
  - Arquitectura .NET
---

# ADR y Decisiones Arquitectónicas

Un ADR registra decisiones importantes y sus consecuencias.

```md
# ADR-001 - Usar SQL Server

## Contexto
Necesitamos transacciones fuertes e integración con ecosistema Microsoft.

## Decisión
Usaremos SQL Server.

## Consecuencias
Buen soporte en Azure, costo/licenciamiento a considerar.
```

## Checklist

- [ ] Documento decisiones costosas.
- [ ] Incluyo alternativas.
- [ ] Escribo consecuencias negativas.
- [ ] Reviso ADRs cuando cambia el contexto.
