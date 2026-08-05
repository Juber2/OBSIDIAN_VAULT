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

# Clean Architecture pragmatica

## Dependencias

Domain no depende de nadie. Application depende de Domain. Infrastructure implementa detalles. API orquesta entrada/salida.

## Estructura


```text
src/
  App.Api/
  App.Application/
  App.Domain/
  App.Infrastructure/
```


## Regla real

Clean Architecture vale si protege negocio y permite testear casos de uso. Si solo agrega carpetas, no ayuda.

## Flujo

Controller -> UseCase/Handler -> Domain -> Repository interface -> Infrastructure EF implementation.

## Errores comunes

- Entidades dependiendo de DbContext
- Application devolviendo IActionResult
- Crear interfaces para todo sin necesidad

## Practica

- [ ] Crear diagrama de dependencias del proyecto

## Relacionado

- [[Arquitectura dotNET]]
- [[NET - Roadmap]]
