---
tags:
  - NET
  - Fundamentos
Creado: 2026-05-21
Relacionado:
  - .NET Fundamentos
---

# Plataforma .NET, CLR, SDK y Runtime

.NET es una plataforma para crear aplicaciones web, APIs, servicios, consola, desktop, cloud y mobile. C# es el lenguaje más común dentro de ese ecosistema.

## Piezas principales

| Pieza | Función |
|---|---|
| SDK | Permite crear, compilar, testear y publicar proyectos |
| Runtime | Ejecuta aplicaciones ya compiladas |
| CLR | Common Language Runtime: memoria, GC, excepciones, tipos |
| IL | Intermediate Language generado al compilar |
| JIT | Compila IL a código máquina en ejecución |

## Comandos base

```bash
dotnet --info
dotnet new console -n Demo
dotnet build
dotnet run
dotnet test
```

## Senior note

Cuando algo falla en producción, entender runtime, GC, threads y configuración pesa más que memorizar sintaxis.

## Checklist

- [ ] Sé qué instala el SDK.
- [ ] Entiendo qué ejecuta el runtime.
- [ ] Sé que C# compila a IL.
- [ ] Puedo crear un proyecto desde terminal.
