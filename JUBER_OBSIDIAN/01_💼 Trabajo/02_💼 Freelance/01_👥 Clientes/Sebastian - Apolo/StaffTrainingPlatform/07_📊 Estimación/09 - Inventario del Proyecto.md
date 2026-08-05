---
tags: [inventario, analisis, stafftraining]
fecha: 2026-06-10
fuente: Análisis código React/Figma
archivos_analizados: 51+
lineas_codigo: "~10,000+"
---

# 🗂️ Inventario del Proyecto

> Análisis exhaustivo del diseño React/Figma existente. Base sobre la que se construyó la estimación.

---

## Estadísticas del código fuente

| Métrica | Valor |
|---|---|
| Archivos `.tsx` analizados | 51+ |
| Líneas de código totales | ~10,000+ |
| Páginas implementadas (completas) | 19 |
| Rutas registradas (incluyendo sin implementar) | 35+ |
| Componentes UI primitivos (Shadcn/Radix) | 30+ |
| Componentes custom | 10+ |
| Contextos (Auth) | 1 |
| Layouts | 3 |

---

## Sistema de Roles

| Rol | Email mock | Acceso |
|---|---|---|
| `superadmin` | super@capaactiva.com | Gestión global de la plataforma |
| `company` | admin@empresa.com | Gestión de su empresa |
| `employee` | empleado@empresa.com | Experiencia de aprendizaje |
| `support` | apoyo@capaactiva.com | Acceso controlado por empresa asignada |

---

## Inventario de Páginas

### Módulo SuperAdmin

| Página | Estado | Complejidad | Archivo |
|---|---|---|---|
| Dashboard SuperAdmin | ✅ Implementada | 🟠 Alta | `NewDashboard.tsx` |
| Listado de Empresas | ✅ Implementada | 🟡 Media | `Companies.tsx` |
| Crear Empresa | ✅ Implementada | 🟡 Media | `CreateCompany.tsx` |
| Editar Empresa | ✅ Implementada | 🟡 Media | `EditCompany.tsx` |
| Cursos Plantilla | ✅ Implementada | 🟢 Baja | `TemplateCourses.tsx` |
| Usuarios de Apoyo | ✅ Implementada | 🔴 Ultra | `SAUsers.tsx` |
| Crear Usuario Apoyo | ✅ Implementada | 🔴 Ultra | `SACreateUser.tsx` |
| Roles de Apoyo | ✅ Implementada | 🟠 Alta | `SARoles.tsx` |
| Crear Rol Apoyo | ✅ Implementada | 🟠 Alta | `SACreateRole.tsx` |
| Support Dashboard | ⚠️ Parcial | 🟡 Media | (por definir) |

### Módulo Company

| Página | Estado | Complejidad | Archivo |
|---|---|---|---|
| Dashboard | ✅ Implementada | 🟡 Media | `Dashboard.tsx` |
| Listado Cursos | ✅ Implementada | 🟡 Media | `Courses.tsx` |
| Crear Curso | ✅ Implementada | 🟡 Media | `CreateCourse.tsx` |
| Editar Curso | ✅ Implementada | 🟡 Media | `EditCourse.tsx` |
| CourseManage | ✅ Implementada | 🔴 Ultra | `CourseManage.tsx` (~2,450 líneas) |
| Seguimiento de Curso | ✅ Implementada | 🟠 Alta | `CourseTracking.tsx` |
| Diseñador Certificados | ✅ Implementada | 🟠 Alta | `CertificateDesigner.tsx` |
| Gestión de Áreas | ✅ Implementada | 🟢 Baja | `Areas.tsx` |
| Gestión de Empleados | ✅ Implementada | 🟠 Alta | `Employees.tsx` |
| Roles de Empresa | ✅ Implementada | 🟢 Baja | `Roles.tsx` |
| Usuarios Empresa | ⚠️ Ruta existe | 🟡 Media | `Users.tsx` |
| Calendario | ⚠️ Ruta existe | 🟡 Media | `Calendar.tsx` |
| Plazos | ⚠️ Ruta existe | 🟡 Media | `Deadlines.tsx` |
| Contenido Curso | ⚠️ Ruta existe | 🟡 Media | `CourseContent.tsx` |
| Editor de Examen | ⚠️ Ruta existe | 🟠 Alta | `ExamEditor.tsx` |
| Crear Evaluación | ⚠️ Ruta existe | 🟡 Media | `CreateEvaluation.tsx` |
| Evaluaciones | ⚠️ Ruta existe | 🟡 Media | `Evaluations.tsx` |
| Asignaciones | ⚠️ Ruta existe | 🟡 Media | `Assignments.tsx` |
| Configuración | ⚠️ Ruta existe | 🟡 Media | `Settings.tsx` |
| Crear Rol | ⚠️ Ruta existe | 🟡 Media | `CreateRole.tsx` |
| Editar Rol | ⚠️ Ruta existe | 🟡 Media | `EditRole.tsx` |

### Módulo Employee

| Página | Estado | Complejidad | Archivo |
|---|---|---|---|
| Dashboard | ✅ Implementada | 🟡 Media | `Dashboard.tsx` |
| Mis Cursos | ✅ Implementada | 🟠 Alta | `MyCourses.tsx` |
| Vista de Curso | ✅ Implementada | 🔴 Ultra | `CourseView.tsx` |
| Resultado Examen | ⚠️ Parcial | 🟠 Alta | `ExamResult.tsx` |
| Vista Certificado | ⚠️ Parcial | 🟡 Media | `CertificateView.tsx` |
| Mis Evaluaciones | ⚠️ Ruta existe | 🟡 Media | (por implementar) |
| Mis Certificados | ⚠️ Parcial | 🟡 Media | `Certificates.tsx` |
| Mi Progreso | ⚠️ Ruta existe | 🟡 Media | (por implementar) |

---

## Componentes Reutilizables

### Custom

| Componente | Descripción | Usado en |
|---|---|---|
| `WireframeTable` | Tabla configurable con title, description, toolbar | Todos los módulos |
| `WireframeButton` | Botón con variantes | Todos los módulos |
| `WireframeInput` | Input con label y validación | Formularios |
| `WireframeSelect` | Select con label | Formularios |
| `WireframeBox` | Container con label | CourseManage |
| `AssignCourseModal` | Modal bidireccional asignación cursos | Company, Employee |
| `NotificationCenter` | Centro de notificaciones (bell + dropdown) | Todos |
| `Header` | Header global con usuario | Todos |
| `Sidebar` | Sidebar con navegación por módulo | Todos |
| `RouterErrorBoundary` | Manejo de errores de rutas | App |

### UI Primitivos (Shadcn/Radix)

```
Accordion, Alert, AlertDialog, AspectRatio, Avatar,
Badge, Breadcrumb, Button, Calendar, Card, Carousel,
Chart, Checkbox, Collapsible, Command, ContextMenu,
Dialog, Drawer, DropdownMenu, Form, HoverCard, Input,
InputOTP, Label, Menubar, NavigationMenu, Pagination,
Popover, Progress, RadioGroup, ResizablePanels,
ScrollArea, Select, Separator, Sheet, Sidebar, Skeleton,
Slider, Sonner (Toast), Switch, Table, Tabs, Textarea,
ToggleGroup, Toggle, Tooltip
```

---

## Rutas del Sistema

```
/                               → Redirect a Login
/login                          → Pantalla de Login

── SuperAdmin ──────────────────────────────────────
/superadmin                     → Dashboard SuperAdmin
/superadmin/companies           → Listado de empresas
/superadmin/companies/new       → Crear empresa
/superadmin/companies/:id/edit  → Editar empresa
/superadmin/template-courses    → Cursos plantilla
/superadmin/sa-users            → Usuarios de apoyo
/superadmin/sa-users/new        → Crear usuario apoyo
/superadmin/sa-roles            → Roles de apoyo
/superadmin/sa-roles/new        → Crear rol apoyo
/superadmin/support-dashboard   → Dashboard soporte

── Company ─────────────────────────────────────────
/company                        → Dashboard company
/company/courses                → Listado de cursos
/company/courses/new            → Crear curso
/company/courses/:id/edit       → Editar curso
/company/courses/:id/manage     → Gestión completa (multi-tab)
/company/courses/:id/tracking   → Seguimiento del curso
/company/courses/:id/content    → Contenido del curso
/company/courses/:id/exam/new   → Crear evaluación
/company/courses/:id/certificate → Diseñador certificado
/company/areas                  → Gestión de áreas
/company/employees              → Gestión de empleados
/company/calendar               → Calendario
/company/deadlines              → Plazos
/company/roles                  → Roles de empresa
/company/users                  → Usuarios de empresa
/company/settings               → Configuración

── Employee ────────────────────────────────────────
/employee                       → Dashboard empleado
/employee/my-courses            → Mis cursos
/employee/courses/:id           → Ver curso
/employee/courses/:id/exam      → Tomar examen
/employee/courses/:id/result    → Resultado del examen
/employee/courses/:id/certificate → Ver certificado
/employee/evaluations           → Mis evaluaciones
/employee/certificates          → Mis certificados
/employee/progress              → Mi progreso
```

---

## Features Especiales Identificadas

| Feature | Dónde se usa | Complejidad técnica |
|---|---|---|
| Drag & Drop lecciones | CourseManage → Tab Contenido | 🟠 Alta |
| Drag & Drop bloques de contenido | CourseManage → Editor lección | 🟠 Alta |
| Editor de bloques (5 tipos) | CourseManage | 🟠 Alta |
| Diseñador certificados en tiempo real | CertificateDesigner, CourseManage | 🟠 Alta |
| Banco de preguntas aleatorio | CourseManage → Tab Evaluación | 🟡 Media |
| Modal asignación bidireccional | Courses, Employees | 🟡 Media |
| Matriz permisos 6×5 por empresa | SACreateUser, SACreateRole | 🔴 Ultra |
| Multi-tenancy (datos por empresa) | Todo el sistema BE | 🔴 Ultra |
| Impersonation (ingresar como empresa) | Companies list | 🟠 Alta |
| Acciones masivas sobre empleados | Employees | 🟡 Media |
| Timer de examen | ExamView | 🟡 Media |
| Video embed YouTube/Vimeo | CourseView, CourseManage | 🟡 Media |
| Generación PDF certificados | CertificateView, Employee | 🟠 Alta |

---

## 🔗 Ver también

- [[01 - Resumen Ejecutivo]]
- [[03 - Módulo SuperAdmin]]
- [[04 - Módulo Company]]
- [[05 - Módulo Employee]]
