---
tags:
  - dotnet
  - Intermedio
  - efcore
Creado: 2026-05-29
Nivel: Intermedio
Estado: Pendiente
Prioridad: Alta
Relacionado:
  - Datos EF Core SQL
  - NET
---

# Migrations y cambios de esquema

## Migration

Representa cambio versionado de schema. Debe revisarse como codigo, especialmente drops, renames y cambios de tipo.

## Comandos


```bash
dotnet ef migrations add AddUsers
dotnet ef database update
```


## Produccion

No ejecutes migraciones a ciegas en startup si el sistema es crítico. Genera scripts, revisa locks y prepara rollback.

## Datos existentes

Cambios no-null, renames y splits requieren estrategia de backfill.

## Errores comunes

- Borrar migration aplicada sin pensar
- Cambiar columnas grandes en hora pico
- No revisar SQL generado

## Practica

- [ ] Crear migration y revisar script SQL

## Relacionado

- [[Datos EF Core SQL]]
- [[NET - Roadmap]]
