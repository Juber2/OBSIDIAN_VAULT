---
tags:
  - NET
  - Docker
Creado: 2026-05-21
Relacionado:
  - DevOps con .NET
---

# Docker con .NET

Docker empaqueta la app y su runtime.

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

## Checklist

- [ ] Uso imagen runtime para ejecutar.
- [ ] No incluyo secretos.
- [ ] Uso `.dockerignore`.
- [ ] Expongo health checks.
