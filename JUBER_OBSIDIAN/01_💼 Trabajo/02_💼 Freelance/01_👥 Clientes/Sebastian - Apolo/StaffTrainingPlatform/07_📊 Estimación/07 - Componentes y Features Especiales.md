---
tags: [estimacion, componentes, features, stafftraining]
fecha: 2026-06-10
fe_total: 152h
be_total: 50h
total: 202h
---

# 🧩 Componentes Shared & Features Especiales

> Componentes reutilizables entre módulos y funcionalidades transversales que requieren integración especializada.

**Total: FE 152h / BE 50h = 202h**

---

## Componentes Compartidos (Angular)

Estos componentes se usan en múltiples módulos y se desarrollan una vez.

### Tabla Genérica con Búsqueda

Equivalente al `WireframeTable` de React, pero potenciado para Angular.

**Funcionalidades:**
- Columnas configurables con `render` personalizado
- Búsqueda integrada (input con debounce)
- Header con título opcional + badge contador + descripción opcional
- Si no hay título ni descripción: buscador se alinea a la derecha
- Paginación del lado cliente / servidor
- Estado vacío configurable (icono + mensaje)
- Loading skeleton
- Selección múltiple (checkboxes)

| Tarea | Horas |
|---|---|
| Componente tabla base con columnas tipadas | 10h |
| Input de búsqueda integrado con layout inteligente | 6h |
| Paginación cliente + servidor | 6h |
| Selección múltiple + barra acciones | 6h |
| Loading skeleton + empty state | 4h |
| **Subtotal** | **32h** |

---

### Componentes de Formulario

Equivalentes a `WireframeInput`, `WireframeSelect`, `WireframeButton`.

| Componente | Descripción | Horas |
|---|---|---|
| `AppInput` | Input con label, validación, error message | 4h |
| `AppSelect` | Select con opciones tipadas | 4h |
| `AppTextarea` | Textarea con contador de caracteres | 3h |
| `AppButton` | Botón con variantes (primary, secondary, danger, ghost) + loading | 4h |
| `AppColorPicker` | Color picker nativo con preview | 4h |
| `AppFileUpload` | Upload con preview, validación tipo/tamaño, progress bar | 10h |
| **Subtotal** | **29h** |

---

### Modal de Asignación de Cursos (Bidireccional)

**Dos modos:**
1. `from-course` → seleccionar empleados/áreas para asignar al curso
2. `from-employee` → seleccionar cursos para asignar al empleado

**Funcionalidades:**
- Tabs: Empleados / Áreas (modo from-course)
- Búsqueda dentro del modal
- Selección múltiple con checkboxes
- Badge contador de seleccionados
- Confirmación con resumen

| Tarea | Horas |
|---|---|
| Modal base + dos modos + tabs | 12h |
| Búsqueda + selección múltiple | 8h |
| **Subtotal** | **20h** |

---

### Centro de Notificaciones

| Tarea | Horas |
|---|---|
| Panel notificaciones (bell icon + dropdown) | 8h |
| Marcado como leída + contador badge | 4h |
| **Subtotal** | **12h** |

---

### Pipes y Directivas

| Pipe / Directiva | Uso | Horas |
|---|---|---|
| `statusBadge` | Estado → clase CSS + texto | 2h |
| `truncate` | Texto largo → truncado con tooltip | 2h |
| `relativeDate` | Fecha → "hace 2 días" | 2h |
| `initials` | "Juan Pérez" → "JP" | 1h |
| `ClickOutside` directive | Cerrar dropdowns al hacer click fuera | 2h |
| **Subtotal** | **9h** |

**Total Componentes Shared: 102h**

---

## Features Especiales Transversales

### Drag & Drop (Angular CDK)

Usado en `CourseManage` para reordenar lecciones y bloques de contenido.

| Tarea | FE | BE |
|---|---|---|
| DnD para reordenar lecciones (CDK DragDrop) | 8h | — |
| DnD para reordenar bloques dentro de lección | 8h | — |
| API: guardar nuevo orden (PATCH con array de IDs) | — | 6h |
| **Subtotal** | **16h** | **6h** |

---

### Integración de Video (YouTube / Vimeo)

Usado en `CourseManage` (editor) y `CourseView` (player).

| Tarea | FE | BE |
|---|---|---|
| Embed YouTube/Vimeo via iframe (URL parsing) | 6h | — |
| Preview thumbnail al pegar URL | 4h | — |
| Player con controles de progreso (tiempo visto) | 6h | — |
| API: guardar tiempo visto por lección | — | 4h |
| **Subtotal** | **16h** | **4h** |

---

### Generación de PDF — Certificados

**Dos enfoques (usar ambos según caso):**

**Client-side** (para vista previa y descarga directa):
- html2canvas → jsPDF
- El HTML del certificado se convierte a imagen y luego a PDF

**Server-side** (para generación consistente, envío por email):
- QuestPDF (.NET) — declarativo, sin dependencias externas
- Genera PDF desde los datos del certificado + config de diseño

| Tarea | FE | BE |
|---|---|---|
| Setup html2canvas + jsPDF | 4h | — |
| Función exportar componente Angular a PDF | 6h | — |
| Setup QuestPDF en .NET | — | 6h |
| Template PDF del certificado (QuestPDF) | — | 14h |
| API: POST /certificates/{id}/pdf → retorna archivo | — | 6h |
| **Subtotal** | **10h** | **26h** |

---

### Sistema de Upload de Archivos

Usado para: logos, firmas, sellos, watermarks, archivos de lección, videos.

| Tarea | FE | BE |
|---|---|---|
| Componente upload genérico (ya contado en shared) | — | — |
| Validación client-side (tipo MIME, tamaño máx) | 4h | — |
| Progress bar de subida | 3h | — |
| Preview de imagen/video después de subir | 3h | — |
| API: POST /files/upload → retorna URL | — | 6h |
| Integración Azure Blob Storage / S3 | — | 12h |
| Servicio limpieza de archivos huérfanos | — | 4h |
| **Subtotal** | **10h** | **22h** |

---

### Notificaciones en Tiempo Real (SignalR)

Usado para: nueva asignación de curso, examen disponible, certificado emitido.

| Tarea | FE | BE |
|---|---|---|
| Setup cliente SignalR en Angular | 6h | — |
| Integración con Centro de Notificaciones | 4h | — |
| Setup Hub SignalR en .NET | — | 8h |
| Envío de notificación por eventos (curso asignado, cert emitido) | — | 10h |
| **Subtotal** | **10h** | **18h** |

---

## Resumen

### Componentes Shared

| Componente | Horas FE |
|---|---|
| Tabla genérica con búsqueda | 32h |
| Componentes de formulario | 29h |
| Modal asignación cursos | 20h |
| Centro de notificaciones | 12h |
| Pipes y directivas | 9h |
| **Total** | **102h** |

### Features Especiales

| Feature | FE | BE | Total |
|---|---|---|---|
| Drag & Drop (CDK) | 16h | 6h | 22h |
| Integración Video | 16h | 4h | 20h |
| Generación PDF | 10h | 26h | 36h |
| Upload de Archivos | 10h | 22h | 32h |
| SignalR Notificaciones | 10h | 18h | 28h |
| **Total** | **62h** | **76h** | **138h** |

---

## 🔗 Ver también

- [[04 - Módulo Company]] ← usa DnD, video, PDF
- [[05 - Módulo Employee]] ← usa video player, PDF
- [[06 - Setup e Infraestructura]] ← storage, SignalR setup
