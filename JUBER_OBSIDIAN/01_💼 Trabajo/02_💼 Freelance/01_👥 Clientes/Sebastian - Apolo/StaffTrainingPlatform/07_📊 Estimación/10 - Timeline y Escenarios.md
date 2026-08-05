---
tags: [estimacion, timeline, cronograma, equipo]
fecha: 2026-06-10
---

# 📅 Timeline y Escenarios de Equipo

---

## Totales base

| | FE Angular | BE .NET | Total |
|---|---|---|---|
| Horas base | 1,483h | 757h | 2,240h |
| Buffer +20% | +297h | +151h | +448h |
| **Con buffer** | **1,780h** | **908h** | **~2,688h** |

---

## ⭐ Escenario 0 — Tú solo (1 dev FE + BE)

> **Solo trabajando en el proyecto.**

### Opción A: Sequential (Recomendado)

Primero completar BE, luego FE. Evita cambio de contexto.

| Fase | Horas | Días (8h/día) | Semanas | 
|---|---|---|---|
| 1️⃣ Backend .NET | 908h | 113 días | 23 sem |
| 2️⃣ Frontend Angular | 1,780h | 222 días | 45 sem |
| 3️⃣ Testing + Deploy | 200h | 25 días | 5 sem |
| **TOTAL SEQUENTIAL** | **2,888h** | **360 días** | **~73 semanas** |

**Tiempo total: ~18 meses** (si trabajas lunes-viernes)

> ⚠️ Con esta estrategia:
> - Terminas BE en mes 5–6 (lanzamiento MVP beta para testing interno)
> - Terminas FE en mes 16–17
> - Testing + fixes en mes 17–18

---

### Opción B: Intercalado (Context switching)

Balancear FE y BE cada día (4h BE + 4h FE). Menos eficiente pero permite entregar más rápido.

| Trabajo | Horas | Días @ 4h/día | Semanas |
|---|---|---|---|
| FE Angular | 1,780h | 445 días | 89 sem |
| BE .NET | 908h | 227 días | 45 sem |
| **Paralelo (ambos a la vez)** | — | **445 días** | **89 sem** |
| **Con ineficiencia +30%** | +270h | **578 días** | **116 sem** |

**Tiempo total: ~22 meses** (si trabajas 8h/día divididas)

> ⚠️ Context switching entre Angular y .NET cada 4 horas es **muy ineficiente**.

---

### 🏆 Recomendación: **Sequential approach — 18 meses**

**Razones:**
1. ✅ Sin context switching → mejor productividad
2. ✅ Lanzas MVP con BE + basic FE en mes 6–7
3. ✅ Puedes tomar feedback en paralelo mientras haces FE
4. ✅ Testing incremental desde el inicio

**Breakdown de 18 meses:**
```
Meses 1–2:   Backend setup + arquitectura       (Sprint 1–2)
Meses 3–4:   Backend APIs: SuperAdmin           (Sprint 3–4)
Meses 5–6:   Backend APIs: Company + Employee   (Sprint 5–7)
Meses 7–8:   Frontend setup + SuperAdmin        (Sprint 8–9)
Meses 9–13:  Frontend: Company (CourseManage x3 sprints)  (Sprint 10–14)
Meses 14–16: Frontend: Employee                 (Sprint 15–17)
Meses 17–18: Testing, bug fixing, deploy        (Sprint 18–20)
```

---

## Escenario A — 1 dev FE + 1 dev BE

> Equipo mínimo absoluto.

| Dev | Horas | Días (8h/día) | Semanas | Meses |
|---|---|---|---|---|
| 1× FE Angular | 1,780h | 222 días | 44 sem | ~11 meses |
| 1× BE .NET | 908h | 113 días | 23 sem | ~5.5 meses |
| **En paralelo** | | | **~44 sem** | **~11 meses** |

> ❌ No recomendado. El FE es el cuello de botella duro. Timeline inaceptable para este scope.

---

## Escenario B — 2 devs FE + 1 dev BE ✅

> **Mínimo recomendado.** Balance entre costo y tiempo.

| Dev | Horas | Días | Semanas | Meses |
|---|---|---|---|---|
| 2× FE Angular | 1,780h ÷ 2 = 890h | 111 días | 22 sem | ~5.5 meses |
| 1× BE .NET | 908h | 113 días | 23 sem | ~5.5 meses |
| **En paralelo** | | | **~23 sem** | **~6 meses** |

> ✅ BE y FE terminan casi al mismo tiempo. Ideal si el cliente quiere entregar en ~6 meses.

---

## Escenario C — 2 devs FE + 2 devs BE + 1 QA ⭐

> **Equipo óptimo.** BE termina antes y el QA valida en paralelo.

| Dev | Horas | Días | Semanas | Meses |
|---|---|---|---|---|
| 2× FE Angular | 890h | 111 días | 22 sem | ~5.5 meses |
| 2× BE .NET | 908h ÷ 2 = 454h | 57 días | 12 sem | ~3 meses |
| 1× QA | (incluido en buffer) | — | — | — |
| **En paralelo** | | | **~22 sem** | **~5–5.5 meses** |

> ⭐ BE termina en sprint 6 y puede hacer revisión de seguridad, optimización y documentación mientras FE finaliza. QA trabaja en flujos E2E desde sprint 8 en adelante.

---

## Escenario D — 3 devs FE + 2 devs BE + 1 QA

> Para cliente que necesita entregar en 4 meses.

| Dev | Horas | Días | Semanas | Meses |
|---|---|---|---|---|
| 3× FE Angular | 1,780h ÷ 3 = 593h | 74 días | 15 sem | ~3.5 meses |
| 2× BE .NET | 454h | 57 días | 12 sem | ~3 meses |
| **En paralelo** | | | **~16 sem** | **~4 meses** |

> Requiere muy buena coordinación. El tercer FE debe ser senior para no generar deuda técnica.

---

## Cronograma detallado — Escenario B (2FE + 1BE) — 22 Sprints

> Sprint = 2 semanas

| Sprint | Semanas | Frontend (2 devs) | Backend (1 dev) |
|---|---|---|---|
| **Sprint 1** | 1–2 | Setup Angular, routing, layouts, auth guard | Setup .NET, Clean Architecture, EF Core, BD |
| **Sprint 2** | 3–4 | Layout SuperAdmin, sidebar, header, interceptors | JWT auth, multi-tenancy middleware, seed roles |
| **Sprint 3** | 5–6 | SA: Dashboard, Listado empresas | APIs: auth, empresas CRUD |
| **Sprint 4** | 7–8 | SA: Crear/Editar empresa | APIs: crear empresa + admin, soft-delete |
| **Sprint 5** | 9–10 | SA: Listado usuarios apoyo, tabla | APIs: usuarios apoyo CRUD, modelo permisos BD |
| **Sprint 6** | 11–12 | SA: Formulario usuario (grid empresas + matriz permisos) | APIs: guardar permisos por (usuario, empresa, módulo) |
| **Sprint 7** | 13–14 | SA: Roles apoyo + formulario + matriz | APIs: roles CRUD, cursos plantilla CRUD |
| **Sprint 8** | 15–16 | Company: Dashboard, Cursos lista, Crear/Editar curso | APIs: cursos empresa, importar plantilla, áreas |
| **Sprint 9** | 17–18 | Company: CourseManage Tab Info + Tab Contenido (lecciones) | APIs: lecciones CRUD, reordenar, bloques, upload archivos |
| **Sprint 10** | 19–20 | Company: CourseManage DnD bloques + 5 tipos de bloque | APIs: storage archivos (Azure Blob), videos |
| **Sprint 11** | 21–22 | Company: CourseManage Tab Evaluación (preguntas, banco) | APIs: examen config, preguntas CRUD, banco aleatorio |
| **Sprint 12** | 23–24 | Company: CourseManage Tab Certificado + Vista Previa | APIs: configuración certificado, QuestPDF setup |
| **Sprint 13** | 25–26 | Company: Diseñador Certificados standalone | APIs: generar PDF certificado server-side |
| **Sprint 14** | 27–28 | Company: Empleados (tabla, crear, accesos, masivo) | APIs: empleados CRUD, activar acceso, email credenciales |
| **Sprint 15** | 29–30 | Company: Seguimiento cursos (2 tabs), Áreas, Roles | APIs: tracking por empleado y área, métricas |
| **Sprint 16** | 31–32 | Company: Calendario, Plazos, Config, Usuarios empresa | APIs: calendario, configuración empresa |
| **Sprint 17** | 33–34 | Employee: Dashboard, Mis Cursos (3 tabs) | APIs: dashboard employee, cursos por estado |
| **Sprint 18** | 35–36 | Employee: Vista de Curso (sidebar, video, navegación) | APIs: lecciones del curso, marcar progreso |
| **Sprint 19** | 37–38 | Employee: ExamView (timer, preguntas), ExamResult | APIs: iniciar intento, calificar, registrar resultado |
| **Sprint 20** | 39–40 | Employee: Certificados, Mi Progreso, Mis Evaluaciones | APIs: certificados, historial progreso, evaluaciones |
| **Sprint 21** | 41–42 | Componentes shared (tabla, modal asignación, SignalR FE) | APIs: SignalR Hub, notificaciones por evento |
| **Sprint 22** | 43–44 | Bug fixing, pulido UI, E2E tests, deploy staging | Optimización BD, seguridad, índices, deploy |

---

## Hitos clave

| Hito | Sprint | Semana |
|---|---|---|
| 🏁 Autenticación funcionando con 4 roles | Sprint 2 | Sem 4 |
| 🏁 SuperAdmin completo (empresas, usuarios, roles) | Sprint 7 | Sem 14 |
| 🏁 Company: Cursos CRUD básico | Sprint 8 | Sem 16 |
| 🏁 CourseManage completo (4 tabs) | Sprint 12 | Sem 24 |
| 🏁 Company: módulo completo | Sprint 16 | Sem 32 |
| 🏁 Employee: módulo completo | Sprint 20 | Sem 40 |
| 🏁 MVP listo para testing interno | Sprint 21 | Sem 42 |
| 🏁 **Go-live** | Sprint 22 | Sem 44 |

---

## Distribución de horas por sprint (Escenario B)

```
Sprints 1–2:   Setup e infraestructura        ~160h FE + BE
Sprints 3–7:   SuperAdmin completo            ~400h FE + BE
Sprints 8–16:  Company completo               ~840h FE + BE
Sprints 17–20: Employee completo              ~400h FE + BE
Sprints 21–22: Shared, testing, deploy        ~300h FE + BE
─────────────────────────────────────────────────────────
TOTAL                                        ~2,100h neto
                                             ~2,688h con buffer
```

---

## Recomendaciones finales

| # | Recomendación |
|---|---|
| 1 | **Empezar BE desde día 1** — El modelo de BD (multi-tenancy, permisos) es la pieza más crítica y define todo lo demás |
| 2 | **CourseManage en 3 sprints dedicados** — No intentar hacerlo en 1. Es la pantalla más compleja del sistema |
| 3 | **Validar el PDF temprano** — La generación de certificados PDF cross-platform puede ser sorpresivamente compleja |
| 4 | **Diseñar la BD de permisos antes de sprint 5** — La matriz 6×5 por empresa necesita un modelo de datos flexible |
| 5 | **QA desde sprint 10** — No esperar al final para testar. Integrar QA incremental desde que hay flujos funcionales |
| 6 | **Módulos no implementados** (+300h) — Calendar, Deadlines, Evaluations, Assignments requieren definición de diseño antes de estimar |

---

## 🔗 Ver también

- [[01 - Resumen Ejecutivo]]
- [[02 - Supuestos y Metodología]]
- [[00 - Índice]]
