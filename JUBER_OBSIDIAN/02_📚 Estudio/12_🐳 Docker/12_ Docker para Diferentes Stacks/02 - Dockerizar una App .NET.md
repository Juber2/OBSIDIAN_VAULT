---
tags:
  - Docker
  - DevOps
  - Docker-Stacks
Creado: 2026-07-28
Relacionado:
  - Docker para Diferentes Stacks
  - DOCKER
---

# 02 - Dockerizar una App .NET

## Que vas a aprender

En este capitulo vas a aprender a dockerizar una aplicacion **.NET** (como StaffTrainingPlatform, el proyecto sobre el que trabajas), usando las imagenes oficiales del SDK y runtime, y el patron especifico de multi-stage build para proyectos .NET.

## Explicacion

.NET tiene una separacion muy clara entre las imagenes oficiales de **SDK** (`mcr.microsoft.com/dotnet/sdk`, pesada, con todo el tooling de compilacion) y **runtime** (`mcr.microsoft.com/dotnet/aspnet`, mucho mas liviana, solo lo necesario para ejecutar una app ya compilada), lo cual encaja perfectamente con el patron de multi-stage build: la etapa de build usa la imagen del SDK para restaurar paquetes NuGet y compilar (`dotnet publish`), y la etapa final usa la imagen de runtime, mucho mas chica.

Es importante copiar primero los archivos `.csproj` y ejecutar `dotnet restore` antes de copiar el resto del codigo fuente, siguiendo el mismo principio de cache de capas visto en el capitulo de Imagenes: restaurar paquetes NuGet es lento, y no deberia repetirse en cada build si solo cambio el codigo fuente, no las dependencias.

## Modelo mental

Pensa la imagen del SDK como el taller completo de un mecanico (con todas las herramientas de diagnostico y reparacion), y la imagen de runtime como el auto ya terminado saliendo del taller: el cliente final no necesita ni deberia recibir el taller completo, solo el vehiculo ya listo para andar.

## Ejemplo base

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY ["StaffTrainingPlatform.API/StaffTrainingPlatform.API.csproj", "StaffTrainingPlatform.API/"]
RUN dotnet restore "StaffTrainingPlatform.API/StaffTrainingPlatform.API.csproj"
COPY . .
WORKDIR /src/StaffTrainingPlatform.API
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:10.0
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENTRYPOINT ["dotnet", "StaffTrainingPlatform.API.dll"]
```

## Como se usa en un proyecto real

1. Se copian los archivos `.csproj` antes que el resto del codigo, para aprovechar cache de `dotnet restore` entre builds donde solo cambia el codigo fuente.
2. Se usa `dotnet publish -c Release` (no `dotnet build`) para generar el artefacto optimizado de produccion.
3. Se configura el puerto de escucha de Kestrel via variable de entorno (`ASPNETCORE_URLS`) para que coincida con lo publicado en el container.
4. Se usa la variante `-alpine` de las imagenes de .NET cuando la compatibilidad lo permite, para reducir aun mas el tamaño final.
5. Se conecta esta imagen con el pipeline de CI/CD del proyecto, siguiendo el mismo patron visto para NestJS en el tema anterior.

## Criterio junior

Un junior debe poder dockerizar una aplicacion .NET basica con multi-stage build usando las imagenes oficiales de SDK y runtime.

## Criterio mid

Un mid debe aprovechar correctamente el cache de `dotnet restore` copiando primero los `.csproj`, y configurar variables de entorno especificas de ASP.NET Core dentro del container.

## Criterio senior

Un senior debe estandarizar el patron de Dockerfile para todos los servicios .NET del equipo, evaluando imagenes Alpine para .NET y midiendo el impacto real en tamaño y performance.

## Errores comunes

- Copiar todo el codigo fuente antes de restaurar paquetes NuGet, perdiendo cache de restore en cada build donde solo cambio codigo.
- Usar la imagen del SDK completa como imagen final de produccion en vez de la de runtime, mucho mas pesada de lo necesario.
- No configurar correctamente `ASPNETCORE_URLS` o el puerto esperado por Kestrel dentro del container.
- No aplicar usuario no-root (capitulo de Seguridad) a la imagen final de una aplicacion .NET.

## Practica

- [ ] Dockeriza una aplicacion .NET real con multi-stage build usando SDK y runtime oficiales.
- [ ] Verifica que el cache de `dotnet restore` se aprovecha correctamente entre builds con solo cambios de codigo.
- [ ] Configura el puerto de Kestrel para que coincida con el expuesto en el Dockerfile.
- [ ] Aplica un usuario no-root a la imagen final.
- [ ] Compara el tamaño de la imagen final usando runtime estandar vs runtime Alpine.

## Preguntas de repaso

- Por que .NET separa imagenes de SDK y runtime, y como encaja eso con multi-stage builds?
- Que orden de instrucciones aprovecha mejor el cache de `dotnet restore`?
- Que variable de entorno controla en que puerto escucha Kestrel dentro del container?
- Que ventaja de tamaño ofrece la variante Alpine de las imagenes de .NET, y que riesgo de compatibilidad hay que evaluar?
