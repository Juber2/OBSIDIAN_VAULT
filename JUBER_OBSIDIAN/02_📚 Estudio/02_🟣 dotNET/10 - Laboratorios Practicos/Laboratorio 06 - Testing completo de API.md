---
tags:
  - dotnet
  - laboratorio
  - testing
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos dotNET
  - NET
---

# Laboratorio 06 - Testing completo de API

## Objetivo

Cubrir reglas de dominio, casos de uso, endpoints y persistencia con tests apropiados.

## Suite minima

- Unit tests para value objects.
- Unit tests para validators.
- Integration tests para endpoints.
- Tests con DB real para constraints y queries importantes.

## Criterio

No busques 100% coverage. Cubre comportamiento que puede romper negocio: auth, permisos, duplicados, errores y transacciones.

## Estructura sugerida


```text
tests/
  App.UnitTests/
  App.IntegrationTests/
```


## Errores comunes

- Tests que repiten codigo
- Mockear EF sin sentido
- No probar casos negativos

## Practica

- [ ] Crear test de email duplicado
- [ ] Crear test de endpoint protegido
- [ ] Crear test de refresh reusado

## Relacionado

- [[Laboratorios Practicos dotNET]]
- [[NET]]
