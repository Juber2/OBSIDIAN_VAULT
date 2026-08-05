---
tags:
  - dotnet
  - Senior
  - senior
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Senioridad Proyectos dotNET
  - NET
---

# Migracion legacy y refactoring

## Estrategia

Primero caracteriza comportamiento con tests o snapshots. Luego refactor pequeño. Después cambia diseño.

## Strangler fig

Crear rutas nuevas alrededor de legacy e ir migrando casos.

## Riesgo

El refactor sin cobertura en sistemas críticos es apuesta. A veces conviene aislar antes que reescribir.

## Errores comunes

- Big bang rewrite
- Cambiar diseño y comportamiento juntos
- Eliminar código sin entender usuarios

## Practica

- [ ] Planear migracion de controller gigante

## Relacionado

- [[Senioridad Proyectos dotNET]]
- [[NET - Roadmap]]
