---
tags:
  - contexto
  - staff-training-platform
  - capa-activa-corp
Actualizado: 2026-06-15
---

# Contexto del Proyecto — CapaActivaCorp

## Los tres proyectos

| Rol | Stack | Ruta |
|---|---|---|
| Referencia visual (Figma→React) | React 18 + Vite + Tailwind v4 + shadcn/ui + MUI | `C:\JUBER\FREELANCE\SEBASTIAN\DISEÑO_REACT_FIGMA\Capaactivacorpfigma-main` |
| Backend | .NET 10 + Clean Architecture + CQRS + EF Core + SQL Server | `C:\JUBER\FREELANCE\SEBASTIAN\StaffTrainingPlatform` |
| Frontend real | Angular (corre en WSL Ubuntu) | `\\wsl.localhost\Ubuntu\home\juber\projects\capaActivaCorpWeb` |

## Documentación viva (Obsidian)
Base: `F:\OBSIDIAN\JUBER\JUBER_OBSIDIAN\01_💼 Trabajo\02_💼 Freelance\01_👥 Clientes\Sebastian - Apolo\StaffTrainingPlatform`

## Arquitectura Backend
- Capas: Domain → Application (CQRS con MediatR) → Infrastructure → API → Transversal
- Contextos: `/platform` (SuperAdmin global) y `/tenant` (empresa específica)
- Auth: JWT 15 min + Refresh Token 7 días persistido en BD, con rotation, revoke y approve session

## Estado de implementación (2026-06-15)

### Backend — implementado ✅
- Auth: Login, RefreshToken, RevokeSession, ApproveSession
- `POST /platform/companies`, `PUT`, `DELETE`
- `POST /platform/users`, `PUT`, `DELETE`
- `GET /platform/roles`, `GET /platform/roles/{id}`, `GET /platform/permissions/matrix`, `POST`, `PUT`, `DELETE`
- `GET /tenant/roles`, `GET /tenant/roles/{id}`, `POST`, `PUT`, `DELETE`
- `POST /tenant/users`, `PUT`, `DELETE`

### Backend — pendiente ⏳
- `GET /platform/companies`, `GET /platform/companies/{id}`
- `GET /platform/users`, `GET /platform/users/{id}`
- `GET /tenant/users`, `GET /tenant/users/{id}`
- Dashboard APIs (platform)
- Módulos: Cursos, Lecciones, Bloques, Exámenes, Certificados, Empleados, Progreso, Upload

### Frontend Angular — estado
- Scaffolding hecho con features: `platform/`, `tenant/`, `identity/`
- Subcarpetas: `administration/`, `access/`, `learning/`
- Sin integración con APIs aún

### Plantilla React (solo referencia visual)
- Páginas SuperAdmin: Dashboard, Companies, Users, Roles, TemplateCourses
- Páginas Company: Courses, CourseManage, CertificateDesigner, Employees, Areas, Roles, Users, Calendar, Tracking
- Páginas Employee: Dashboard, MyCourses, CourseView, ExamView, ExamResult, Certificates, Progress

## Entidades Domain principales
- `User` (PersonId, Username, Password, IsLocked, LockedUntil, Email)
- `RolePlatform`, `RoleTenant`, `Permission`, `Resource`
- `RolePlatformPermission`, `RoleTenantPermission`
- `RefreshToken` (IP, UserAgent incluidos)

## Repositorios Infrastructure
CompanyRepository, UserRepository, RolePlatformRepository, RoleTenantRepository, PermissionRepository, PersonRepository, RefreshTokenRepository
