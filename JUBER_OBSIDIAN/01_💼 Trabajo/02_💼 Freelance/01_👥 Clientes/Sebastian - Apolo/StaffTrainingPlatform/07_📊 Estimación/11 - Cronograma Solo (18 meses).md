---
tags: [estimacion, timeline-solo, un-developer]
fecha: 2026-06-10
titulo: "Tu proyecto solo — 18 meses"
---

# 📅 Cronograma: TÚ SOLO — 18 Meses

> Plan de trabajo sequential: Backend primero (~6 meses), luego Frontend (~12 meses), testing/deploy (~1.5 meses).

---

## Resumen

| Fase | Duración | Inicio | Fin | Horas |
|---|---|---|---|---|
| **Fase 1: Backend .NET** | 6 meses | Mes 1 | Mes 6 | 908h |
| **Fase 2: Frontend Angular** | 11 meses | Mes 7 | Mes 17 | 1,780h |
| **Fase 3: Testing + Deploy** | 1 mes | Mes 18 | Mes 18 | 200h |
| **TOTAL** | **~18 meses** | — | — | **2,888h** |

---

## Fase 1: Backend .NET — Meses 1–6

> 908 horas ÷ 40h/semana = 23 semanas

### Mes 1–2: Setup + Arquitectura (80h)

| Semana | Tarea | Horas | Avance |
|---|---|---|---|
| 1–2 | Setup Clean Architecture, EF Core, SQL Server, migrations | 40h | ✅ BD lista |
| 3–4 | JWT auth, multi-tenancy middleware, seed data | 40h | ✅ Auth funciona |
| | **Subtotal** | **80h** | |

**Hito:** Puedes hacer login con 4 roles diferentes y acceso controlado por empresa.

---

### Mes 3–4: SuperAdmin APIs (160h)

| Semana | Tarea                                             | Horas    | Avance           |
| ------ | ------------------------------------------------- | -------- | ---------------- |
| 5–6    | CRUD empresas, usuarios admin automático          | 40h      | ✅ Empresas       |
| 7–8    | Dashboard metrics + actividad reciente            | 40h      | ✅ Dashboard      |
| 9–10   | Usuarios de apoyo CRUD + permisos granulares (BD) | 50h      | ✅ Usuarios apoyo |
| 11–12  | Roles de apoyo + matriz permisos model            | 30h      | ✅ Roles          |
|        | **Subtotal**                                      | **160h** |                  |

**Hito:** SuperAdmin completamente funcional (APIs). El modelado de permisos 6×5 está listo.

---

### Mes 5–6: Company + Employee APIs (668h)

| Semana | Tarea | Horas | Avance |
|---|---|---|---|
| 13–14 | CRUD cursos, áreas, importar plantilla | 40h | ✅ Cursos |
| 15–16 | Lecciones, bloques, upload archivos (Azure/S3) | 50h | ✅ Contenido |
| 17–18 | Exámenes: config, preguntas, banco aleatorio, calificación | 60h | ✅ Exámenes |
| 19–20 | Certificados + QuestPDF (generación server-side) | 50h | ✅ Certificados |
| 21–22 | Empleados, acceso, email, tracking | 50h | ✅ Empleados |
| 23 | Progreso, evaluaciones, reportes, SignalR Hub | 50h | ✅ Reportes |
| | **Subtotal** | **300h** | |

**Hito:** Todo el backend listo para consumir desde Angular. MVPβ (solo APIs) listo para testing interno.

---

## Fase 2: Frontend Angular — Meses 7–17

> 1,780 horas ÷ 40h/semana = 44.5 semanas ≈ 11 meses

### Mes 7–8: Setup + SuperAdmin (160h)

| Mes | Tarea | Horas | Avance |
|---|---|---|---|
| 7 | Setup Angular, routing lazy-load, layouts, guards, interceptors | 80h | ✅ Estructura |
| 8 | SuperAdmin: Dashboard, Empresas (CRUD), Cursos plantilla | 80h | ✅ SuperAdmin básico |
| | **Subtotal** | **160h** | |

**Hito:** Puedes navegar SuperAdmin completo, login funciona.

---

### Mes 9–10: SuperAdmin Completo (120h)

| Mes | Tarea | Horas | Avance |
|---|---|---|---|
| 9–10 | Usuarios de apoyo (tabla + formulario) + matriz permisos (6×5 con checkboxes) | 80h | ✅ Usuarios + matriz |
| 9–10 | Roles de apoyo + diseño UI | 40h | ✅ Roles |
| | **Subtotal** | **120h** | |

**Hito:** SuperAdmin 100% funcional. La matriz de permisos está pulida.

---

### Mes 11–14: Company — Cursos + CourseManage (520h)

Este es el bloque más pesado. **Divide en sprints pequeños.**

| Mes | Tarea | Horas | Avance |
|---|---|---|---|
| 11 | Cursos: Dashboard, listado, crear/editar, modal asignación | 80h | ✅ Cursos CRUD |
| 12 | CourseManage — Tab Información + Tab Contenido (lecciones + DnD) | 120h | ✅ Lecciones |
| 13 | CourseManage — DnD bloques, 5 tipos de bloque, upload files | 120h | ✅ Editor bloques |
| 14 | CourseManage — Tab Evaluación (preguntas, banco) + Tab Certificado + Vista Previa | 100h | ✅ Exámenes + Cert |
| 14 | Diseñador de Certificados (standalone) | 100h | ✅ Diseñador |
| | **Subtotal** | **520h** | |

**Hito:** CourseManage completamente funcional (la página más compleja del sistema).

---

### Mes 15–16: Company — Resto (240h)

| Mes | Tarea | Horas | Avance |
|---|---|---|---|
| 15 | Empleados (tabla, crear, accesos, masivo) + Áreas + Roles | 100h | ✅ Empleados |
| 15–16 | Seguimiento de cursos (Por empleado / Por área), Calendario, Plazos | 80h | ✅ Tracking |
| 16 | Configuración empresa, Usuarios empresa, componentes shared | 60h | ✅ Config + Shared |
| | **Subtotal** | **240h** | |

**Hito:** Company 100% funcional.

---

### Mes 17: Employee (360h)

| Mes | Tarea | Horas | Avance |
|---|---|---|---|
| 17 | Dashboard + Mis Cursos (3 tabs) | 80h | ✅ Dashboard |
| 17 | Vista de Curso: player, lecciones, navegación, recurso | 100h | ✅ Player |
| 17 | Sistema Exámenes: ExamView, timer, ExamResult | 100h | ✅ Examen |
| 17 | Certificados, Progreso, Evaluaciones | 80h | ✅ Certs + Reportes |
| | **Subtotal** | **360h** | |

**Hito:** Employee 100% funcional. Sistema completo en Angular.

---

## Fase 3: Testing + Deploy — Mes 18

> 200 horas

| Semana | Tarea | Horas | Avance |
|---|---|---|---|
| Semana 1 | E2E tests: flujos principales (login, crear curso, asignar, tomar examen) | 50h | ✅ E2E |
| Semana 2 | Bug fixes, UI polish, responsive | 80h | ✅ Polish |
| Semana 3 | CI/CD pipeline, Docker, despliegue staging | 40h | ✅ DevOps |
| Semana 4 | Documentación, training, go-live preparación | 30h | ✅ Docs |
| | **Subtotal** | **200h** | |

---

## Hitos clave por mes

| Mes | Hito | Status |
|---|---|---|
| 1–2 | Setup .NET + Auth JWT + Multi-tenancy | 🟢 Backend |
| 3–4 | SuperAdmin APIs completo | 🟢 Backend |
| 5–6 | Company + Employee APIs (lanzamiento MVPβ) | 🟢 Backend ✅ ALPHA |
| 7–8 | Angular setup + SuperAdmin FE | 🔵 Frontend |
| 9–10 | SuperAdmin FE 100% | 🔵 Frontend |
| 11–14 | Company FE (CourseManage 3 meses) | 🔵 Frontend |
| 15–16 | Company FE resto | 🔵 Frontend |
| 17 | Employee FE 100% | 🔵 Frontend ✅ BETA |
| 18 | Testing + Deployment | 🟡 QA ✅ LIVE |

---

## Consejos para trabajar solo

### 1. Backend first (Sprint 0–14)

✅ **Ventajas:**
- No hay bloqueos: mientras haces BE, no necesitas FE
- Lanzas MVPβ (APIs) en mes 6 para testing interno
- Puedes tomar feedback mientras empiezas FE

### 2. DDD (Domain-Driven Design) te ahorra tiempo

✅ Modela bien la BD en sprint 1–2 (permisos, multi-tenancy)
✅ Todo lo demás fluye naturalmente

### 3. Automatiza desde día 1

✅ CI/CD pipeline simple: GitHub Actions + Docker
✅ Tests automáticos en cada push
✅ Evita "surprise bugs" en go-live

### 4. CourseManage — Tu cuello de botella

⚠️ Esta pantalla solo tiene 120 líneas de descripción en el contexto pero internamente es ULTRA compleja.
✅ Dedica los meses 12–13 solo para esto.
✅ Hazlo en subsecciones: lecciones (1 semana) → bloques (1 semana) → examen (1 semana) → certificado (1 semana).

### 5. UI/UX — Reutiliza

✅ Usa Angular Material / PrimeNG (no reinventes).
✅ Template Bootstrap para layouts.
✅ No custom CSS innecesario.

### 6. Testing — Incremental

✅ Unit tests mientras desarrollas (no al final).
✅ E2E tests desde mes 16 (cuando hay flujos completos).
✅ No apuntes a 100% cobertura, enfócate en flujos críticos.

---

## Budget de tiempo por complejidad

| Si algo toma... | Esperado | Plan B | Plan C |
|---|---|---|---|
| 50 horas | 5 días | 1 semana | 2 semanas |
| 100 horas | 2.5 semanas | 3 semanas | 1 mes |
| 150 horas | 1 mes | 5 semanas | 6 semanas |

> Siempre suma 20% extra si es la primera vez haciendo algo.

---

## 🔗 Ver también

- [[10 - Timeline y Escenarios]] (comparativa de todos los escenarios)
- [[02 - Supuestos y Metodología]] (metodología)
- [[09 - Inventario del Proyecto]] (qué exactamente tienes que hacer)
