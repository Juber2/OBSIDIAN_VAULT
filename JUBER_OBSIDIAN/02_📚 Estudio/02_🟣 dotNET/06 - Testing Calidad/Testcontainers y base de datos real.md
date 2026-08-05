---
tags:
  - dotnet
  - Avanzado
  - testing
Creado: 2026-05-29
Nivel: Avanzado
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Testing Calidad dotNET
  - NET
---

# Testcontainers y base de datos real

## Por qué DB real

EF InMemory no se comporta como SQL Server/PostgreSQL. Puede ocultar errores de SQL, transacciones, constraints e includes.

## Testcontainers

Levanta contenedores desechables para integration tests realistas.

## Criterio

Usa DB real para repositories/queries críticas. Para lógica pura, unit tests.

## Errores comunes

- Confiar en InMemory para queries complejas
- Tests lentos sin estrategia
- No limpiar datos

## Practica

- [ ] Probar constraint de email unico con DB real

## Relacionado

- [[Testing Calidad dotNET]]
- [[NET - Roadmap]]
