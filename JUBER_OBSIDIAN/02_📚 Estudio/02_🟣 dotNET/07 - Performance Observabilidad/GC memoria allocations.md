---
tags:
  - dotnet
  - Senior
  - performance
Creado: 2026-05-29
Nivel: Senior
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Performance Observabilidad dotNET
  - NET
---

# GC memoria allocations

## GC

El garbage collector libera memoria administrada. No significa que puedas asignar sin costo. Muchas allocations aumentan presión de GC.

## Generaciones

Objetos cortos mueren joven. Objetos largos llegan a generaciones superiores y cuestan más. Large Object Heap requiere cuidado.

## Buenas señales

Reducir strings temporales, evitar LINQ caliente en paths críticos, usar pooling cuando hay evidencia.

## Herramientas

dotnet-counters, dotnet-trace, dotnet-dump, Visual Studio profiler, PerfView.

## Errores comunes

- Optimizar micro sin perfil
- Usar Span en codigo normal por moda
- Ignorar memory leaks por eventos o static caches

## Practica

- [ ] Medir allocations de endpoint de listado

## Relacionado

- [[Performance Observabilidad dotNET]]
- [[NET - Roadmap]]
