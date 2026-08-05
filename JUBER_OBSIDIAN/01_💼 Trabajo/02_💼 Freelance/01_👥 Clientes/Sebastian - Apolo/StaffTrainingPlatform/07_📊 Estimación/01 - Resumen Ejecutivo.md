---
tags: [estimacion, resumen, stafftraining]
fecha: 2026-06-10
---

# 📋 Resumen Ejecutivo

## Totales por área

| Módulo / Área | FE Angular | BE .NET | Total |
|---|---|---|---|
| [[06 - Setup e Infraestructura\|Setup & Arquitectura]] | 82h | 102h | 184h |
| Autenticación | 28h | 22h | 50h |
| [[03 - Módulo SuperAdmin\|SuperAdmin]] | 254h | 146h | 400h |
| [[04 - Módulo Company\|Company]] | 580h | 244h | 824h |
| [[05 - Módulo Employee\|Employee]] | 272h | 114h | 386h |
| [[07 - Componentes y Features Especiales\|Componentes Shared]] | 96h | — | 96h |
| [[07 - Componentes y Features Especiales\|Features Especiales]] | 56h | 50h | 106h |
| [[08 - Testing y DevOps\|Testing]] | 92h | 56h | 148h |
| [[08 - Testing y DevOps\|DevOps]] | 23h | 23h | 46h |
| **Subtotal base** | **1,483h** | **757h** | **2,240h** |
| **+20% contingencia** | +297h | +151h | +448h |
| **TOTAL CON BUFFER** | **1,780h** | **908h** | **~2,688h** |

---

## 🏆 Top 5 páginas más costosas

| # | Página / Feature | FE | BE | Total |
|---|---|---|---|---|
| 1 | CourseManage (editor multi-tab completo) | 170h | 66h | **236h** |
| 2 | Usuarios de apoyo SA + matriz permisos | 88h | 44h | **132h** |
| 3 | Diseñador de Certificados | 86h | 28h | **114h** |
| 4 | Gestión de Empleados (acceso masivo) | 70h | 40h | **110h** |
| 5 | Sistema de Exámenes (employee) | 50h | 40h | **90h** |

---

## 👥 Escenarios de equipo

### ⭐ Escenario 0 — Tú solo (1 dev FE + BE)

| Enfoque | Horas | Días | Meses |
|---|---|---|---|
| Sequential (BE primero, luego FE) | 2,888h | 360 días | **~18 meses** ✅ Recomendado |
| Intercalado (4h BE + 4h FE/día) | 3,158h | 578 días | ~22 meses ⚠️ Ineficiente |

> Mejor estrategia: Termina todo el backend primero, luego frontend. Evita context switching.

---

### Escenario A — 1 FE + 1 BE (mínimo absoluto)

| Dev | Horas | Días | Semanas |
|---|---|---|---|
| 1× Frontend Angular | 1,780h | 222 días | ~44 sem |
| 1× Backend .NET | 908h | 113 días | ~23 sem |
| **En paralelo** | — | — | **~11 meses** |

> ⚠️ No recomendado para este scope — timeline demasiado largo.

---

### Escenario B — 2 FE + 1 BE ✅ (mínimo recomendado)

| Dev | Horas | Días | Semanas |
|---|---|---|---|
| 2× Frontend Angular | 1,780h ÷ 2 = 890h | 111 días | ~22 sem |
| 1× Backend .NET | 908h | 113 días | ~23 sem |
| **En paralelo** | — | — | **~6 meses** |

> ✅ El backend es ligeramente el cuello de botella. Viable si BE arranca en semana 1.

---

### Escenario C — 2 FE + 2 BE + 1 QA ⭐ (óptimo)

| Dev | Horas | Días | Semanas |
|---|---|---|---|
| 2× Frontend Angular | 1,780h ÷ 2 = 890h | 111 días | ~22 sem |
| 2× Backend .NET | 908h ÷ 2 = 454h | 57 días | ~12 sem |
| 1× QA | Incluido en buffer | — | — |
| **En paralelo** | — | — | **~5–5.5 meses** |

> ⭐ El FE domina. BE termina antes y puede apoyar con integraciones y optimización.

---

## 📅 Cronograma de sprints (Escenario B — 2FE + 1BE)

| Sprint | Semanas | FE | BE |
|---|---|---|---|
| 1–2 | 1–4 | Setup Angular, layouts, auth | Setup .NET, BD, JWT |
| 3–4 | 5–8 | SuperAdmin: dashboard, empresas | APIs empresas, tenants, auth |
| 5–6 | 9–12 | SuperAdmin: usuarios SA, roles SA | APIs usuarios, permisos, roles |
| 7–8 | 13–16 | Company: cursos, crear/editar | APIs cursos, asignaciones |
| 9–11 | 17–22 | Company: CourseManage (3 sprints) | APIs lecciones, bloques, evaluaciones |
| 12–13 | 23–26 | Company: empleados, tracking, cert | APIs empleados, tracking, PDF cert |
| 14–15 | 27–30 | Employee: dashboard, cursos, curso view | APIs progreso, lecciones |
| 16–17 | 31–34 | Employee: exámenes, certificados | APIs exámenes, calificación |
| 18–19 | 35–38 | Employee: progreso, evaluaciones | APIs progreso, reportes |
| 20–22 | 39–44 | Bug fixing, pulido UI, componentes extra | Optimización, seguridad, deploy |

---

## ⚠️ Nota importante sobre módulos incompletos

Las siguientes rutas existen en el sistema pero **no tienen implementación completa** en el diseño actual. No están incluidas en la estimación principal pero suman aproximadamente:

- Calendar, Deadlines, ExamEditor, ExamView, ExamResult completo
- CertificateView completo, MyEvaluations, Certificates, Progress (employee)
- Support Dashboard

> **Estimación adicional**: ~+200h FE / ~+100h BE

---

## 🔗 Ver detalle

- [[03 - Módulo SuperAdmin]]
- [[04 - Módulo Company]]
- [[05 - Módulo Employee]]
- [[10 - Timeline y Escenarios]]
