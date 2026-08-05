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

# Code Review Senior dotNET

## Qué revisar

Seguridad, consistencia HTTP, boundaries, transacciones, performance, observabilidad, test coverage y claridad del dominio.

## Checklist

- ¿El endpoint retorna status correcto?
- ¿Valida tenant y permisos en backend?
- ¿Hay CancellationToken?
- ¿La query materializa tarde?
- ¿No expone entidades EF?
- ¿Errores usan ProblemDetails?
- ¿Hay test para el riesgo principal?

## Comentario bueno

Explica riesgo y alternativa: “Esta query hace Include de todo el grafo para un listado. Proyectemos directo a DTO para evitar columnas y joins innecesarios.”

## Errores comunes

- Reviews solo de estilo
- Pedir patrones sin justificar
- Aprobar auth sin pruebas

## Practica

- [ ] Revisar un controller y una query EF

## Relacionado

- [[Senioridad Proyectos dotNET]]
- [[NET - Roadmap]]
