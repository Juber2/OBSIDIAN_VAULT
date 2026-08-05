---
tags: [tracking, progreso, checklist, stafftraining]
fecha: 2026-06-15
estado: "En progreso"
progreso: 20%
---

# ✅ Checklist de Progreso — 18 Meses

> Marca los checkboxes conforme completes cada tarea. Actualiza `progreso` arriba cuando termines un mes.
> Última actualización: 2026-06-15

---

## 🟢 FASE 1: Backend .NET — Meses 1–6

### Mes 1–2: Setup + Arquitectura (80h)

- [x] Scaffolding solución .NET 10 con Clean Architecture
- [x] Configurar Entity Framework Core + SQL Server
- [x] Diseñar modelo de BD completo (entidades base)
- [x] Crear migraciones iniciales
- [x] Implementar JWT Authentication (generación + validación)
- [x] Setup multi-tenancy middleware
- [x] Seed data de producción (roles base, permisos iniciales)
- [x] Swagger + OpenAPI setup

**Estado: ✅ Completado**

---

### Mes 3–4: SuperAdmin APIs (160h)

#### CRUD Empresas (Platform)
- [x] POST /platform/companies — crear empresa
- [ ] GET /platform/companies — listar empresas paginado
- [ ] GET /platform/companies/{id} — obtener empresa individual
- [x] PUT /platform/companies/{id} — actualizar empresa
- [x] DELETE /platform/companies/{id} — soft-delete empresa
- [ ] POST /platform/companies/{id}/admin — crear usuario admin con empresa

#### Usuarios Platform (SuperAdmin / Apoyo)
- [x] POST /platform/users — crear usuario platform
- [ ] GET /platform/users — listar usuarios platform
- [ ] GET /platform/users/{id} — obtener usuario individual
- [x] PUT /platform/users/{id} — actualizar usuario
- [x] DELETE /platform/users/{id} — eliminar usuario

#### Roles Platform
- [x] GET /platform/roles — listar roles
- [x] GET /platform/roles/{id} — obtener rol individual
- [x] GET /platform/permissions/matrix — matriz de permisos
- [x] POST /platform/roles — crear rol
- [x] PUT /platform/roles/{id} — actualizar rol
- [x] DELETE /platform/roles/{id} — eliminar rol

#### Dashboard APIs (Platform)
- [ ] GET /platform/dashboard/metrics — total empresas, empleados, cursos
- [ ] GET /platform/dashboard/activity — últimos eventos/actividad
- [ ] GET /platform/dashboard/top-companies — ranking por engagement

**Estado: ⏳ En progreso (CRUD parcial completado, faltan GETs de listado y dashboard)**

---

### Mes 3–4 (Tenant): Roles y Usuarios Tenant

#### Roles Tenant
- [x] GET /tenant/roles — listar roles (por companyId)
- [x] GET /tenant/roles/{id} — obtener rol individual
- [x] POST /tenant/roles — crear rol tenant
- [x] PUT /tenant/roles/{id} — actualizar rol tenant
- [x] DELETE /tenant/roles/{id} — eliminar rol tenant

#### Usuarios Tenant
- [x] POST /tenant/users — crear usuario tenant
- [ ] GET /tenant/users — listar usuarios tenant
- [ ] GET /tenant/users/{id} — obtener usuario tenant individual
- [x] PUT /tenant/users/{id} — actualizar usuario tenant
- [x] DELETE /tenant/users/{id} — eliminar usuario tenant

**Estado: ⏳ En progreso (CRUD parcial, faltan GETs de listado)**

---

### Mes 3–4: Auth Avanzada

- [x] POST /auth/login — login con JWT
- [x] POST /auth/refresh-token — refresh token
- [x] POST /auth/revoke-session — revocar sesión
- [x] POST /auth/approve-session — aprobar sesión sospechosa
- [x] Refresh token rotation implementado
- [x] RefreshToken entity + repository persistido en BD
- [x] Detección de sesión sospechosa (IP, User-Agent)

**Estado: ✅ Completado**

---

### Mes 5–6: Company + Employee APIs (300h)

#### Cursos (40h)
- [ ] POST /api/courses — crear curso
- [ ] GET /api/courses — listar cursos de la empresa
- [ ] PUT /api/courses/{id} — actualizar curso
- [ ] DELETE /api/courses/{id} — eliminar curso
- [ ] POST /api/courses/import-template — importar desde plantilla
- [ ] GET /api/courses/{id}/info — obtener info completa del curso

#### Lecciones + Bloques (50h)
- [ ] POST /api/courses/{id}/lessons — crear lección
- [ ] GET /api/courses/{id}/lessons — listar lecciones
- [ ] PUT /api/courses/{id}/lessons/{lessonId} — actualizar
- [ ] DELETE /api/courses/{id}/lessons/{lessonId} — eliminar
- [ ] POST /api/courses/{id}/lessons/{lessonId}/reorder — reordenar lecciones
- [ ] POST /api/courses/{id}/lessons/{lessonId}/blocks — crear bloque
- [ ] PUT /api/courses/{id}/lessons/{lessonId}/blocks/{blockId} — actualizar bloque
- [ ] DELETE /api/courses/{id}/lessons/{lessonId}/blocks/{blockId} — eliminar

#### Upload de Archivos (20h)
- [ ] Integrar Azure Blob Storage o S3
- [ ] POST /api/files/upload — subir archivo y retornar URL
- [ ] Validación tipo MIME y tamaño

#### Exámenes (60h)
- [ ] POST /api/courses/{id}/exam/config
- [ ] GET /api/courses/{id}/exam/config
- [ ] POST /api/courses/{id}/questions
- [ ] GET /api/courses/{id}/questions
- [ ] PUT /api/courses/{id}/questions/{qId}
- [ ] DELETE /api/courses/{id}/questions/{qId}
- [ ] GET /api/courses/{id}/exam/start
- [ ] POST /api/courses/{id}/exam/submit
- [ ] Lógica de banco aleatorio implementada

#### Certificados (50h)
- [ ] POST /api/courses/{id}/certificate/config
- [ ] GET /api/courses/{id}/certificate/config
- [ ] POST /api/certificates/{id}/pdf — generar PDF con QuestPDF

#### Empleados (50h)
- [ ] POST /api/employees — crear empleado
- [ ] GET /api/employees — listar empleados de la empresa
- [ ] PUT /api/employees/{id} — actualizar
- [ ] DELETE /api/employees/{id} — eliminar
- [ ] POST /api/employees/{id}/activate-access
- [ ] POST /api/employees/send-credentials

#### Progreso + Tracking (50h)
- [ ] POST /api/progress/{employeeId}/{courseId}/mark-lesson-complete
- [ ] GET /api/progress/{employeeId}/{courseId}
- [ ] GET /api/courses/{courseId}/tracking/by-employee
- [ ] GET /api/courses/{courseId}/tracking/by-area
- [ ] SignalR Hub setup

**Estado: ◯ Pendiente**

---

## 🔵 FASE 2: Frontend Angular — Meses 7–17

> Frontend: `capaActivaCorpWeb` (Angular, WSL Ubuntu)
> Referencia visual: `Capaactivacorpfigma-main` (React + Vite + shadcn/ui)

### Mes 7–8: Setup + SuperAdmin (160h)

#### Setup Angular
- [x] Scaffolding Angular con estructura modular (features: platform, tenant, identity)
- [x] Lazy loading por módulo
- [ ] Configurar tema global
- [ ] HTTP Client + environments (dev/staging/prod)
- [ ] Interceptors: auth token, error global, loading spinner
- [ ] Auth Guards por rol (CanActivate)
- [ ] Layouts base: SuperAdmin, Company, Employee
- [ ] Servicio de notificaciones (toast/snackbar)
- [ ] Routing principal + página de login

#### SuperAdmin Básico
- [ ] Página de login con validación
- [ ] Pantalla Dashboard SuperAdmin
- [ ] Listado de Empresas
- [ ] Crear Empresa (formulario)
- [ ] Editar Empresa (formulario)
- [ ] Cursos Plantilla

**Estado: ⏳ En progreso (scaffolding hecho, features sin implementar)**

---

### Mes 9–10: SuperAdmin Completo (120h)

- [ ] Listado de usuarios platform (tabla con stats)
- [ ] Formulario crear/editar usuario platform
- [ ] Grid de selección de empresas
- [ ] Panel de permisos por empresa
- [ ] Matriz de permisos (módulos × acciones)
- [ ] Roles platform — listado, crear, editar, eliminar

**Estado: ◯ Pendiente**

---

### Mes 11–14: Company — Cursos + CourseManage (520h)

- [ ] Dashboard Company
- [ ] CRUD Cursos
- [ ] CourseManage — tabs: Información, Contenido, Evaluación, Certificado
- [ ] Editor de bloques con drag & drop
- [ ] Diseñador de Certificados
- [ ] Exámenes con banco aleatorio

**Estado: ◯ Pendiente**

---

### Mes 15–16: Company — Resto (240h)

- [ ] CRUD Empleados + activación de acceso
- [ ] CRUD Áreas + Roles tenant
- [ ] Seguimiento de cursos por empleado/área
- [ ] Calendario + plazos
- [ ] Usuarios tenant + asignación de roles
- [ ] Centro de notificaciones

**Estado: ◯ Pendiente**

---

### Mes 17: Employee (360h)

- [ ] Dashboard Employee
- [ ] Mis Cursos (en progreso, completados, disponibles)
- [ ] Vista de Curso + video player
- [ ] Exámenes (timer, banco aleatorio, resultado)
- [ ] Certificados (vista + descarga PDF)
- [ ] Progreso general

**Estado: ◯ Pendiente**

---

## 🟡 FASE 3: Testing + Deploy — Mes 18

- [ ] E2E Tests (Playwright/Cypress)
- [ ] Responsive design
- [ ] CI/CD GitHub Actions
- [ ] Docker (API + Angular)
- [ ] Deploy staging
- [ ] Documentación + Go-Live

**Estado: ◯ Pendiente**

---

## 📊 Resumen de Progreso

### Fase 1: Backend (Meses 1–6)
- [x] Mes 1–2: Setup + Auth + Arquitectura ✅
- [x] Auth avanzada: JWT, refresh token, sesiones ✅
- [⏳] Mes 3–4: CRUD Platform/Tenant (parcial — faltan GETs de listado)
- [ ] Mes 3–4: Dashboard APIs
- [ ] Mes 5–6: Company + Employee APIs

**Progreso Fase 1: ~35%**

### Fase 2: Frontend (Meses 7–17)
- [⏳] Mes 7–8: Scaffolding Angular hecho, vistas pendientes
- [ ] Mes 9–17: Sin iniciar

**Progreso Fase 2: ~5%**

### Fase 3: Testing (Mes 18)
- [ ] Sin iniciar

**Progreso Fase 3: 0%**

---

## 🔗 Ver también

- [[11 - Cronograma Solo (18 meses)]] ← Descripción detallada de cada fase
- [[04 - Módulo Company]] ← Especificaciones de Company
- [[05 - Módulo Employee]] ← Especificaciones de Employee
- [[Frontend Angular - capaActivaCorpWeb]] ← Estructura del frontend real
- [[Plantilla React Figma - Referencia Visual]] ← Guía de pantallas
