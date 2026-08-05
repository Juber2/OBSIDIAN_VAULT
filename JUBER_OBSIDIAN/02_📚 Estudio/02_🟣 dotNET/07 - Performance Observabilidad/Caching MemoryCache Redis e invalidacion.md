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

# Caching MemoryCache Redis e invalidacion

## MemoryCache

Bueno para datos pequeños, locales y no críticos. En múltiples instancias, cada app tiene cache distinta.

## Redis

Cache distribuida para escenarios multi-instancia, sesiones, locks o datos compartidos.

## Invalidacion

La parte difícil no es guardar, es saber cuándo borrar o refrescar.

## Patrones

Cache-aside: buscar cache, si falta ir a DB, guardar, devolver.

## Errores comunes

- Cachear permisos sin invalidar
- TTL enorme por comodidad
- Cachear datos por usuario con key incompleta

## Practica

- [ ] Diseñar cache para catálogo de cursos

## Relacionado

- [[Performance Observabilidad dotNET]]
- [[NET - Roadmap]]
