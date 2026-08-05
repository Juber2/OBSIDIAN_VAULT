---
tags: [estimacion, setup, infraestructura, arquitectura]
fecha: 2026-06-10
fe_total: 110h
be_total: 124h
total: 234h
---

# ⚙️ Setup e Infraestructura

> Comprende toda la base del proyecto: arquitectura, autenticación, configuración de entornos y el esqueleto sobre el que se construyen los módulos.

**Total: FE 110h / BE 124h = 234h**

---

## Frontend — Angular

### Estructura del proyecto

```
src/
├── app/
│   ├── core/                  # Guards, interceptors, services base
│   ├── shared/                # Componentes y pipes reutilizables
│   ├── features/
│   │   ├── superadmin/        # Módulo lazy
│   │   ├── company/           # Módulo lazy
│   │   └── employee/          # Módulo lazy
│   └── auth/                  # Login, guards
├── environments/
└── assets/
```

### Tareas de Setup FE

| Tarea | Horas |
|---|---|
| Scaffolding Angular 17+ con estructura modular | 8h |
| Configuración Lazy Loading por módulo (superadmin, company, employee) | 8h |
| Setup Angular Material / PrimeNG + tema global | 12h |
| HTTP Client + Environment configs (dev/staging/prod) | 6h |
| **Interceptors**: Auth token, Error global, Loading spinner | 10h |
| **Auth Guards**: CanActivate por rol (superadmin, company, employee) | 12h |
| Layout SuperAdmin (Sidebar + Header + Content) | 8h |
| Layout Company (Sidebar + Header + Content) | 6h |
| Layout Employee (Sidebar + Header + Content) | 6h |
| Servicio de notificaciones (toast/snackbar) | 6h |
| Servicio HTTP base (genérico con tipado) | 8h |
| Configuración ESLint + Prettier | 4h |
| **Subtotal FE Setup** | **94h** |

### Autenticación FE

| Tarea | Horas |
|---|---|
| Pantalla Login (reactive form + validación) | 12h |
| Auth Service (login, logout, token storage, decode JWT) | 10h |
| Redirect al módulo correcto por rol | 6h |
| **Subtotal FE Auth** | **28h** |

**Total FE: 94h + 28h = 122h** *(incluye algo de overlap con layouts)*

---

## Backend — .NET

### Arquitectura elegida: Clean Architecture

```
Solution/
├── Domain/           # Entidades, Value Objects, interfaces
├── Application/      # CQRS (Commands/Queries), DTOs, Validators
├── Infrastructure/   # EF Core, Repositories, Servicios externos
└── API/              # Controllers, Middlewares, DI setup
```

### Diseño de Base de Datos

#### Entidades principales

```sql
-- Multi-tenancy
Empresa (Id, Nombre, RUC, Email, Teléfono, Estado, Plan, LímiteEmpleados, Notas)

-- Usuarios
Usuario (Id, EmpresaId?, Nombre, Email, PasswordHash, Rol, Estado, ÚltimoAcceso)
RolSoporte (Id, Nombre, Código, Descripción)
UsuarioSoporte (Id, UsuarioId, RolSoporteId)
PermisoUsuarioEmpresa (Id, UsuarioId, EmpresaId, Módulo, Acción, Permitido)

-- Cursos
Curso (Id, EmpresaId, Nombre, Descripción, Categoría, Estado, Duración, SecuencialObligatorio, EmiteCertificado)
Lección (Id, CursoId, Título, Duración, Orden)
BloqueContenido (Id, LecciónId, Tipo, Contenido, Orden)

-- Empleados
Empleado (Id, EmpresaId, UsuarioId?, Nombre, Email, ÁreaId, Estado)
Área (Id, EmpresaId, Nombre)

-- Asignaciones y Progreso
AsignaciónCurso (Id, CursoId, EmpleadoId?, ÁreaId?, FechaLímite)
ProgresoCurso (Id, EmpleadoId, CursoId, Estado, FechaCompletado, Calificación)
ProgresoLección (Id, EmpleadoId, LecciónId, Completada, FechaCompletado)

-- Evaluaciones
ConfigExamen (Id, CursoId, Duración, PorcentajeMínimo, Intentos, MezclarPreguntas, MostrarResultados, UsaBanco, PreguntasPorIntento)
Pregunta (Id, CursoId, Texto, Tipo, Puntos)
Opción (Id, PreguntaId, Texto, EsCorrecta)
IntentoExamen (Id, EmpleadoId, CursoId, Inicio, Fin, Score, Aprobó, NúmeroIntento)
RespuestaIntento (Id, IntentoId, PreguntaId, OpciónesSeleccionadas)

-- Certificados
ConfigCertificado (Id, CursoId, Plantilla, Colores JSON, Fuente, Borde, Elementos JSON)
Certificado (Id, EmpleadoId, CursoId, FechaObtención, Score, ArchivoPath)

-- Plantillas Globales
CursoPlantilla (Id, Nombre, Categoría, Estado, Descripción, CreadoPor)
```

### Tareas de Setup BE

| Tarea | Horas |
|---|---|
| Scaffolding solución Clean Architecture (.NET 8) | 8h |
| Configuración EF Core + SQL Server / PostgreSQL | 6h |
| **Diseño y modelado de BD** (todas las entidades) | 20h |
| Migraciones iniciales + seed data (roles, permisos base) | 8h |
| **JWT Authentication** (generación, validación, refresh) | 16h |
| Middleware de autorización por rol + claims | 8h |
| **Multi-tenancy middleware** (resolución de empresa por claims) | 12h |
| Repositorio base genérico + Unit of Work | 10h |
| AutoMapper setup + perfiles base | 6h |
| **Middleware global de errores** (Problem Details RFC 7807) | 6h |
| Logging (Serilog + structured logging) | 6h |
| Swagger / OpenAPI + ejemplos | 8h |
| CORS + configuración entornos | 6h |
| Health checks endpoint | 4h |
| **Subtotal BE Setup** | **124h** |

### Autenticación BE

| Tarea | Horas |
|---|---|
| POST /auth/login — validar credenciales + generar JWT | 8h |
| POST /auth/refresh — renovar token | 6h |
| POST /auth/logout — invalidar refresh token | 4h |
| GET /auth/me — datos del usuario autenticado | 4h |
| **Subtotal BE Auth** | **22h** |

> *Auth BE incluido en el setup total para no duplicar*

---

## Configuración de Entornos

| Entorno | Descripción |
|---|---|
| **Development** | BD local, logs verbose, CORS abierto |
| **Staging** | BD staging, logs info, secrets en Azure Key Vault |
| **Production** | BD producción, logs warning+, CDN, HTTPS forzado |

---

## 🔗 Ver también

- [[02 - Supuestos y Metodología]]
- [[08 - Testing y DevOps]]
- [[01_🏗️ Arquitectura]]
