---
tags:
  - frontend
  - angular
  - staff-training-platform
Actualizado: 2026-06-15
---

# Frontend Angular — capaActivaCorpWeb

## Datos del proyecto

| Campo | Valor |
|---|---|
| Ruta | `\\wsl.localhost\Ubuntu\home\juber\projects\capaActivaCorpWeb` |
| Framework | Angular (última versión con standalone components) |
| Entorno | WSL Ubuntu |
| Referencia visual | [[Plantilla React Figma - Referencia Visual]] |

## Estructura de carpetas

```
src/
├── app/
│   ├── features/
│   │   ├── platform/        ← Módulo SuperAdmin (gestión global)
│   │   │   ├── administration/
│   │   │   └── access/
│   │   ├── tenant/          ← Módulo Company (gestión por empresa)
│   │   │   ├── administration/
│   │   │   ├── access/
│   │   │   └── learning/
│   │   └── identity/        ← Auth / Login
│   ├── core/                ← Guards, interceptors, servicios globales
│   ├── shared/              ← Componentes compartidos
│   ├── app.routes.ts
│   ├── app.config.ts
│   └── app.ts
├── environments/
└── styles.css
```

## Mapeo de módulos Angular ↔ API Backend

| Feature Angular | Contexto API | Endpoints relacionados |
|---|---|---|
| `platform/administration` | Platform | `/platform/companies`, `/platform/users` |
| `platform/access` | Platform | `/platform/roles`, `/platform/permissions/matrix` |
| `tenant/administration` | Tenant | `/tenant/users` |
| `tenant/access` | Tenant | `/tenant/roles` |
| `tenant/learning` | Tenant | Cursos, lecciones, exámenes (pendiente BE) |
| `identity` | Auth | `/auth/login`, `/auth/refresh-token` |

## Estado actual (2026-06-15)

- [x] Scaffolding base con estructura de features
- [x] Módulos separados: platform, tenant, identity
- [ ] Interceptor de autenticación JWT
- [ ] Auth Guards por rol
- [ ] Layouts diferenciados por contexto
- [ ] Integración con endpoints de Platform
- [ ] Integración con endpoints de Tenant
- [ ] Vistas funcionales

## Notas importantes

- El frontend corre en WSL (Ubuntu), no en Windows directamente.
- La guía de qué pantallas construir está en [[Plantilla React Figma - Referencia Visual]].
- Los roles en el sistema: **SuperAdmin** (platform), **Company Admin** (tenant), **Employee** (tenant).
