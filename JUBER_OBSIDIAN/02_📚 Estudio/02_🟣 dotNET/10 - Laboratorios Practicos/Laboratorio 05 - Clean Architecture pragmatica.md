---
tags:
  - dotnet
  - laboratorio
  - arquitectura
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos dotNET
  - NET
---

# Laboratorio 05 - Clean Architecture pragmatica

## Objetivo

Reorganizar la API en capas sin sobreingenieria: API, Application, Domain e Infrastructure.

## Reglas de dependencia

API referencia Application. Application referencia Domain. Infrastructure referencia Application y Domain. Domain no referencia nada del resto.

## Caso de uso

Crear `CreateUserCommand` o `CreateUserUseCase`, validar reglas y persistir mediante abstraccion clara.

## ADR

Escribe una decision corta explicando por que esa arquitectura vale para este tamaño de app y que costo introduce.

## Errores comunes

- Interfaces para todo sin razon
- Domain dependiendo de EF
- Application devolviendo IActionResult

## Practica

- [ ] Crear solution multi-proyecto
- [ ] Validar dependencias
- [ ] Escribir ADR

## Relacionado

- [[Laboratorios Practicos dotNET]]
- [[NET]]
