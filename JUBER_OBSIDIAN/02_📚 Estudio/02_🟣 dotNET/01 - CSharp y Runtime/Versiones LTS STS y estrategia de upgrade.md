---
tags:
  - dotnet
  - runtime
  - versiones
Creado: 2026-05-30
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - CSharp y Runtime dotNET
  - NET
---

# Versiones LTS STS y estrategia de upgrade

## Estado actual recomendado

Para una base moderna conviene estudiar con .NET 10 como LTS vigente. Microsoft publica una version major cada noviembre; las versiones pares son LTS y las impares STS. .NET 10 fue publicado el 11 de noviembre de 2025 y tiene soporte hasta el 14 de noviembre de 2028 segun la politica oficial.

## LTS vs STS

LTS prioriza estabilidad y soporte largo. STS permite adoptar features antes, pero exige moverse mas rapido. Para sistemas empresariales, LTS suele ser el default salvo que haya una razon tecnica fuerte.

## Estrategia de upgrade

No esperes al fin de soporte. Mantén paquetes actualizados, activa analyzers, ejecuta tests en CI y revisa breaking changes por release. Haz upgrade primero en servicios pequeños o apps internas.

## Checklist de version

- TargetFramework definido y conocido.
- SDK fijado con global.json si el equipo lo necesita.
- Dependencias NuGet revisadas.
- Docker images actualizadas.
- CI usando la misma version que desarrollo.
- Plan de rollback si es produccion.

## Fuentes oficiales

- https://dotnet.microsoft.com/platform/support/policy
- https://dotnet.microsoft.com/platform/support/policy/dotnet-core
- https://learn.microsoft.com/dotnet/core/whats-new/dotnet-10/overview

## Errores comunes

- Subir version sin correr tests
- Usar previews en produccion sin necesidad
- Ignorar fin de soporte

## Practica

- [ ] Revisar version SDK/runtime local
- [ ] Crear global.json para un proyecto sandbox

## Relacionado

- [[CSharp y Runtime dotNET]]
- [[NET]]
