---
tags:
  - dotnet
  - Basico
  - csharp
Creado: 2026-05-29
Nivel: Basico
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - CSharp y Runtime dotNET
  - NET
---

# Plataforma dotNET CLR SDK Runtime

## Piezas principales

| Pieza | Rol |
|---|---|
| SDK | Compila, crea proyectos y trae CLI |
| Runtime | Ejecuta apps ya compiladas |
| CLR | Maneja memoria, JIT, excepciones y tipos |
| BCL | Libreria base: string, IO, collections, tasks |
| NuGet | Paquetes y versionado |

## Flujo de ejecucion

C# se compila a IL. En runtime, el JIT convierte IL a codigo maquina segun plataforma. Ese modelo explica por que .NET es multiplataforma sin perder performance.

## csproj minimo


```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```


## Errores comunes

- Instalar runtime cuando necesitas SDK
- No fijar TargetFramework
- No entender que cada paquete puede traer dependencias transitivas

## Practica

- [ ] Crear proyecto y ejecutar dotnet --info

## Relacionado

- [[CSharp y Runtime dotNET]]
- [[NET - Roadmap]]
