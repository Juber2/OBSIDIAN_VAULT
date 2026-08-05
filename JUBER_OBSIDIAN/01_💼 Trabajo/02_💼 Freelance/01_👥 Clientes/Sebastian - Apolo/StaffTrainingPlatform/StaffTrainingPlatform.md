---
tags:
  - proyecto
  - freelance
  - staff-training-platform
  - capa-activa-corp
Actualizado: 2026-06-15
---

# StaffTrainingPlatform

## Resumen

Proyecto para una plataforma multi-tenant de capacitación de personal para **CapaActivaCorp**.

El sistema separa dos contextos principales:

- **Platform**: administración global de la plataforma (SuperAdmin).
- **Tenant**: operación dentro de una empresa/tenant específico (Company Admin, Employee).

## Tres proyectos que forman el producto

| Rol | Proyecto | Ruta |
|---|---|---|
| Diseño / Referencia visual | React + Vite + shadcn/ui (plantilla Figma) | `C:\JUBER\FREELANCE\SEBASTIAN\DISEÑO_REACT_FIGMA\Capaactivacorpfigma-main` |
| Backend | .NET 10 + Clean Architecture + CQRS | `C:\JUBER\FREELANCE\SEBASTIAN\StaffTrainingPlatform` |
| Frontend real | Angular (WSL Ubuntu) | `\\wsl.localhost\Ubuntu\home\juber\projects\capaActivaCorpWeb` |

## Índice del proyecto

| Área | Nota |
|---|---|
| Contexto general | [[Contexto del Proyecto]] |
| Arquitectura | [[Arquitectura StaffTrainingPlatform]] |
| Frontend Angular | [[Frontend Angular - capaActivaCorpWeb]] |
| Plantilla React/Figma | [[Plantilla React Figma - Referencia Visual]] |
| Decisión Auth/API | [[ADR-001 - Autenticación JWT y API Multi-Tenant]] |
| Decisión Tokens | [[ADR-002 - Manejo de Access Token y Refresh Token]] |
| Seguridad Tokens | [[ADR-003 - Seguridad Avanzada de JWT y Refresh Tokens]] |
| Backend Auth | [[Backend - Autenticación JWT]] |
| API Multi-Tenant | [[Backend - API Multi-Tenant]] |
| Frontend Auth | [[Frontend - Manejo de JWT]] |
| Base de datos | [[Modelo base de datos - CapaActivaCorp]] |
| Checklist progreso | [[12 - Checklist de Progreso]] |
| Pendientes técnicos | [[Pendientes Técnicos]] |

## Estado actual (2026-06-15)

### Backend .NET — implementado
- ✅ Autenticación JWT completa (login, refresh, revoke, approve session)
- ✅ CRUD parcial Platform: companies (sin GETs de listado), users (sin GETs), roles (completo)
- ✅ CRUD parcial Tenant: roles (completo), users (sin GETs)
- ⏳ Pendiente: GETs de listado/detalle para companies y users
- ⏳ Pendiente: Dashboard APIs, módulo Cursos, Empleados, Certificados

### Frontend Angular — scaffolding
- ✅ Estructura de features: `platform/`, `tenant/`, `identity/`
- ⏳ Sin integración con APIs aún

## Decisiones activas

- JWT como mecanismo de autenticación entre frontend y backend.
- Estrategia Access Token de 15 minutos + Refresh Token de 7 días.
- El cliente no anticipa expiración leyendo `exp`; reacciona al `401`.
- El refresh token se persiste en base de datos para permitir revocación.
- Refresh token rotation con detección de reuso.
- Detección de uso sospechoso por IP, User-Agent.
- Email de notificaciones en `User`, no en `Person`.
- Login como único endpoint público.
- El tenant se obtiene desde el JWT, nunca desde el body.
- Separación explícita de endpoints por contexto: `/tenant` y `/platform`.
- JSON en camelCase usando configuración global de ASP.NET Core.

## Pendientes inmediatos

- Completar GETs de listado/detalle: `/platform/companies`, `/platform/users`, `/tenant/users`.
- Definir almacenamiento del token en frontend.
- Definir policies de autorización y conectar con permisos en BD.
- Integrar `AppPermissions.cs` en el pipeline de autorización.
