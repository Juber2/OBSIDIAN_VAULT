---
tags:
  - frontend
  - react
  - figma
  - referencia
  - staff-training-platform
Actualizado: 2026-06-15
---

# Plantilla React / Figma — Referencia Visual

## Datos del proyecto

| Campo | Valor |
|---|---|
| Ruta | `C:\JUBER\FREELANCE\SEBASTIAN\DISEÑO_REACT_FIGMA\Capaactivacorpfigma-main` |
| Framework | React 18 + Vite + Tailwind v4 |
| UI Components | shadcn/ui (Radix) + MUI |
| Router | React Router v7 |
| Propósito | **Referencia visual** — define pantallas que el frontend Angular debe implementar |

> Esta plantilla NO es el frontend real. Es el diseño de referencia generado desde Figma para guiar el desarrollo Angular.

## Stack de la plantilla

```json
React 18 + Vite 6
Tailwind CSS v4
shadcn/ui (Radix UI completo)
MUI (Material UI) v7
React Router v7
Recharts (gráficos)
React Hook Form
React DnD (drag & drop)
```

## Pantallas por rol

### SuperAdmin (`/pages/superadmin/`)

| Archivo | Pantalla |
|---|---|
| `Dashboard.tsx` / `NewDashboard.tsx` | Dashboard principal |
| `Companies.tsx` | Listado de empresas |
| `CreateCompany.tsx` | Crear empresa |
| `EditCompany.tsx` | Editar empresa |
| `SAUsers.tsx` | Usuarios platform |
| `SACreateUser.tsx` | Crear usuario platform |
| `SARoles.tsx` | Roles platform |
| `SACreateRole.tsx` | Crear rol platform |
| `TemplateCourses.tsx` | Cursos plantilla |
| `SupportDashboard.tsx` | Dashboard de soporte |

### Company Admin (`/pages/company/`)

| Archivo | Pantalla |
|---|---|
| `Dashboard.tsx` | Dashboard empresa |
| `Courses.tsx` | Listado de cursos |
| `CreateCourse.tsx` | Crear curso |
| `EditCourse.tsx` | Editar curso |
| `CourseManage.tsx` | Gestionar curso (tabs) |
| `CourseContent.tsx` | Editor de contenido/bloques |
| `CourseTracking.tsx` | Seguimiento de cursos |
| `Evaluations.tsx` | Listado de evaluaciones |
| `CreateEvaluation.tsx` | Crear evaluación |
| `ExamEditor.tsx` | Editor de examen |
| `CertificateDesigner.tsx` | Diseñador de certificados |
| `Employees.tsx` | Listado de empleados |
| `Users.tsx` | Usuarios de la empresa |
| `Roles.tsx` | Roles tenant |
| `CreateRole.tsx` | Crear rol tenant |
| `EditRole.tsx` | Editar rol tenant |
| `Areas.tsx` | Áreas de la empresa |
| `Assignments.tsx` | Asignaciones de cursos |
| `Calendar.tsx` | Calendario |
| `Deadlines.tsx` | Plazos |
| `Settings.tsx` | Configuración empresa |

### Employee (`/pages/employee/`)

| Archivo | Pantalla |
|---|---|
| `Dashboard.tsx` | Dashboard empleado |
| `MyCourses.tsx` | Mis cursos |
| `CourseView.tsx` | Vista de un curso |
| `MyEvaluations.tsx` | Mis evaluaciones |
| `ExamView.tsx` | Tomar examen |
| `ExamResult.tsx` | Resultado del examen |
| `Certificates.tsx` | Mis certificados |
| `CertificateView.tsx` | Ver certificado individual |
| `Progress.tsx` | Mi progreso |

## Componentes compartidos (`/components/`)

- `Header.tsx`, `Sidebar.tsx` — layout base
- `NotificationCenter.tsx` — centro de notificaciones
- `AssignCourseModal.tsx` — modal asignación cursos
- Componentes Wireframe: `WireframeBox`, `WireframeButton`, `WireframeInput`, `WireframeSelect`, `WireframeTable`
- Carpeta `ui/` — componentes shadcn
- Carpeta `figma/` — componentes directos del diseño Figma

## Cómo usar esta referencia

1. Abrir la pantalla React correspondiente para ver el diseño visual
2. Ejecutar con `npm run dev` (o `pnpm dev`) desde `Capaactivacorpfigma-main`
3. Replicar la pantalla en Angular en el feature correspondiente
4. Ver [[Frontend Angular - capaActivaCorpWeb]] para el mapeo de rutas Angular
