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

# Docker y configuracion por ambiente

## Dockerfile base


```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:10.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "App.Api.dll"]
```


## Config

Usa appsettings por ambiente, variables de entorno y secret manager/key vault para secretos.

## Health checks

Necesarios para orquestadores y monitoreo. Distingue liveness y readiness si aplica.

## Errores comunes

- Copiar secretos al image
- No usar multi-stage build
- No fijar puertos/health checks

## Practica

- [ ] Dockerizar API sandbox

## Relacionado

- [[Cloud DevOps Mensajeria dotNET]]
- [[NET - Roadmap]]
