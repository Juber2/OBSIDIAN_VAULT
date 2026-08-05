---
tags:
  - dotnet
  - Avanzado
  - arquitectura
Creado: 2026-05-29
Nivel: Avanzado
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Arquitectura dotNET
  - NET
---

# Modular Monolith y boundaries

## Modular monolith

Una sola app desplegable con modulos internos fuertes. Suele ser mejor paso antes de microservicios.

## Modulo

Debe tener API pública, persistencia propia o schema claro, y reglas de dependencia.

## Ejemplo


```text
Modules/
  Identity/
    PublicApi/
    Application/
    Domain/
    Infrastructure/
  Learning/
  Billing/
```


## Ventaja

Menos complejidad operacional que microservicios, pero más orden que monolito por capas globales.

## Errores comunes

- Llamar modular a carpetas sin boundaries
- Compartir tablas sin reglas
- Dependencias circulares entre modulos

## Practica

- [ ] Diseñar modulos Identity y Learning

## Relacionado

- [[Arquitectura dotNET]]
- [[NET - Roadmap]]
