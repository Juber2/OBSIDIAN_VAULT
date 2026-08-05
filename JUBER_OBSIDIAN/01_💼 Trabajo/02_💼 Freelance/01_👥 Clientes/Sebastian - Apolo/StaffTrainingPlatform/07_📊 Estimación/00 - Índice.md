---
tags: [estimacion, indice, stafftraining]
fecha: 2026-06-10
proyecto: StaffTrainingPlatform
stack: Angular + .NET
---

# 📊 Estimación — StaffTrainingPlatform

> Estimación de desarrollo para migración/implementación del sistema **StaffTrainingPlatform** en stack **Angular (Frontend)** + **.NET (Backend)**.
> Basada en análisis exhaustivo del diseño Figma/React existente.

---

## 🗂️ Contenido de esta carpeta

| Archivo | Descripción |
|---|---|
| [[01 - Resumen Ejecutivo]] | Totales, timeline y escenarios de equipo |
| [[02 - Supuestos y Metodología]] | Bases del cálculo, definiciones, nivel del equipo |
| [[03 - Módulo SuperAdmin]] | Estimación detallada del módulo SuperAdmin |
| [[04 - Módulo Company]] | Estimación detallada del módulo Company |
| [[05 - Módulo Employee]] | Estimación detallada del módulo Employee |
| [[06 - Setup e Infraestructura]] | Arquitectura base, auth, CI/CD, DevOps |
| [[07 - Componentes y Features Especiales]] | Componentes shared, drag&drop, PDF, video |
| [[08 - Testing y DevOps]] | Estrategia de testing + despliegue |
| [[09 - Inventario del Proyecto]] | Inventario completo de páginas y funcionalidades |
| [[10 - Timeline y Escenarios]] | Cronogramas por escenario de equipo |
| [[11 - Cronograma Solo (18 meses)]] | Tu cronograma mes a mes (trabajando solo) |
| **[[12 - Checklist de Progreso]]** | ✅ **Checklist interactivo para ir marcando avance** |

---

## ⚡ Resumen rápido

| Concepto | Valor |
|---|---|
| **Si trabajas solo** | **~18 meses** (backend primero, luego frontend) |
| Total horas base | ~2,240h |
| Total con buffer +20% | ~2,688h |
| Frontend Angular | ~1,780h |
| Backend .NET | ~908h |
| Equipo mínimo recomendado | 2 FE + 1 BE → ~6 meses |
| Equipo óptimo | 2 FE + 2 BE + QA → ~5 meses |

---

## 🧩 Módulos del sistema

```
StaffTrainingPlatform
├── 🔐 Autenticación (4 roles)
├── 👑 SuperAdmin — Gestión global de tenants
├── 🏢 Company — Gestión de contenido educativo
└── 👤 Employee — Experiencia de aprendizaje
```

---

## 🔗 Links útiles

- [[StaffTrainingPlatform]] ← Nota principal del proyecto
- [[01_🏗️ Arquitectura]] ← Decisiones de arquitectura
- [[02_🧩 Backend]] ← Backend notes
- [[03_🅰️ Frontend]] ← Frontend notes
