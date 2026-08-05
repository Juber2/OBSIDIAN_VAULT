---
tags:
  - dotnet
  - Senior
  - devops
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Cloud DevOps Mensajeria dotNET
  - NET
---

# CI CD build test publish

## Pipeline mínimo

Restore, build, test, publish, scan opcional y deploy. Fallar rápido si tests o análisis fallan.

## Calidad

Agrega format/analyzers cuando el equipo los acepte. No conviertas CI en castigo lento.

## Artefactos

Publica binarios o imagen Docker versionada con commit SHA.

## Errores comunes

- Deploy manual sin trazabilidad
- No ejecutar tests en CI
- Usar latest sin version

## Practica

- [ ] Diseñar pipeline para API .NET

## Relacionado

- [[Cloud DevOps Mensajeria dotNET]]
- [[NET - Roadmap]]
