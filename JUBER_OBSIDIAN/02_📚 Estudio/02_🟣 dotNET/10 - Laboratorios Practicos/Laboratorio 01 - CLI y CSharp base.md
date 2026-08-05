---
tags:
  - dotnet
  - laboratorio
  - csharp
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Laboratorios Practicos dotNET
  - NET
---

# Laboratorio 01 - CLI y CSharp base

## Objetivo

Crear una app de consola que modele usuarios, cursos e inscripciones usando C# moderno, records, enums, LINQ, nullability y async simulado.

## Entregables

- Proyecto console con `dotnet new console`.
- Modelos `User`, `Course`, `Enrollment`.
- Value object `Email`.
- Consultas LINQ para activos, inscritos y cursos populares.
- Servicio async falso con CancellationToken.

## Criterios de aceptacion

El proyecto compila sin nullable warnings importantes, no usa `dynamic`, no usa `async void`, y tiene al menos tres consultas LINQ legibles.

## Retrospectiva

Anota que conceptos fueron faciles, cuales confundieron y que refactor harias si esto creciera.

## Errores comunes

- Hacer todo en Program.cs sin estructura
- Ignorar warnings
- No nombrar modelos con intención

## Practica

- [ ] Crear repositorio local
- [ ] Ejecutar dotnet build
- [ ] Escribir README con decisiones

## Relacionado

- [[Laboratorios Practicos dotNET]]
- [[NET]]
