---
tags:
  - pendientes
  - staff-training-platform
Actualizado: 2026-06-15
---

# Pendientes Técnicos

## Autenticación

- [x] Definir expiración del access token: 15 minutos.
- [x] Decidir si habrá refresh token: sí, 7 días.
- [x] Definir tabla para persistir refresh tokens → `RefreshToken` entity + `RefreshTokenRepository`.
- [x] Endpoint `POST /auth/refresh-token` → implementado.
- [x] Implementar refresh token rotation → `RevokeSession` + `ApproveSession` commands.
- [x] Implementar detección de reuso de refresh token.
- [x] Guardar IP, User-Agent y fecha de login original en `RefreshToken`.
- [x] Implementar `ApproveSession` para links de aprobación/revocación.
- [ ] Definir almacenamiento del token en frontend (localStorage vs cookie httpOnly).
- [ ] Definir logout y limpieza de sesión en frontend.
- [ ] Enviar email ante país o dispositivo nuevo.
- [ ] Revocar automáticamente ante geolocalización imposible.
- [ ] Aplicar rate limiting en login y refresh.
- [ ] Definir límite de sesiones concurrentes.

## Autorización

- [ ] Definir policies por contexto (Platform vs Tenant).
- [ ] Definir matriz de permisos por rol → estructura en BD ya existe (`Permission`, `Resource`, `RolePlatformPermission`, `RoleTenantPermission`).
- [ ] Conectar policies en middleware con permisos desde BD.
- [ ] Definir comportamiento de `401` y `403` en frontend.
- [ ] Implementar `AppPermissions.cs` (archivo presente en API pero pendiente de integrar).

## API — Pendientes de implementar

### Platform
- [ ] GET /platform/companies — listar empresas
- [ ] GET /platform/companies/{id} — detalle empresa
- [ ] GET /platform/users — listar usuarios platform
- [ ] GET /platform/users/{id} — detalle usuario platform
- [ ] GET /platform/dashboard/metrics
- [ ] GET /platform/dashboard/activity

### Tenant
- [ ] GET /tenant/users — listar usuarios tenant
- [ ] GET /tenant/users/{id} — detalle usuario tenant

### General
- [ ] Definir paginación estándar para listados.
- [ ] Definir formato de errores unificado.
- [ ] Definir versionado de API.

## Seguridad

- [ ] Agregar rate limiting en login.
- [ ] Definir bloqueo por intentos fallidos (campo `IsLocked` + `LockedUntil` en `User` ya existen).
- [ ] Definir auditoría por tenant.

## Módulos pendientes de construir (Backend)

- [ ] Módulo Cursos (Course, Lesson, Block, ExamConfig, Question)
- [ ] Módulo Certificados (CertificateConfig, CertificatePDF con QuestPDF)
- [ ] Módulo Empleados (Employee, Area, acceso al sistema)
- [ ] Módulo Progreso/Tracking (Progress, LessonCompletion)
- [ ] Upload archivos (Azure Blob / S3)
- [ ] SignalR para notificaciones en tiempo real

## Frontend Angular (capaActivaCorpWeb)

- [ ] Interceptores HTTP (auth token, error global, loading spinner)
- [ ] Auth Guards por rol
- [ ] Layouts base por contexto (SuperAdmin, Company, Employee)
- [ ] Integrar todos los endpoints de Platform (companies, users, roles)
- [ ] Integrar todos los endpoints de Tenant (users, roles)
- [ ] Ver [[Frontend Angular - capaActivaCorpWeb]] para detalle
