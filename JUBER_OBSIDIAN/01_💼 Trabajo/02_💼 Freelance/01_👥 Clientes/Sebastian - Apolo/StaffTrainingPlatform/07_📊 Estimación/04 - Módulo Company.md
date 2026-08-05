---
tags: [estimacion, company, stafftraining]
fecha: 2026-06-10
modulo: Company
fe_total: 580h
be_total: 244h
total: 824h
---

# 🏢 Módulo Company

> Administración de contenido educativo para el administrador de cada empresa. El módulo más extenso del sistema.

**Total módulo: FE 580h / BE 244h = 824h**

---

## Rol que accede

- `company` — administrador de la empresa

---

## Páginas del módulo

### 1. Dashboard Company 🟡 Media complejidad

#### Pantallas / Componentes
- Hero banner con accesos rápidos (Gestión, Seguimiento, Ver Cursos)
- Grid 4 stats: Total empleados, Cursos, Cursos activos, Áreas
- Panel "Actividad Reciente" (col-span-3)
- Panel "Acciones Rápidas" — 4 botones de navegación (col-span-2)

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Hero + botones de acción rápida | 8h | — |
| Stats grid + panels actividad y acciones | 14h | — |
| API: stats de la empresa autenticada | — | 8h |
| API: actividad reciente de la empresa | — | 4h |
| **Subtotal** | **22h** | **12h** |

---

### 2. Gestión de Cursos 🟡 Media complejidad

#### Pantallas / Componentes
- Tabla de cursos: Nombre + N° lecciones, Estado badge
- 4 acciones por fila: Gestión, Seguimiento, Asignar, Eliminar
- Header: botones "Importar Plantilla" y "Crear Curso"
- **Modal AssignCourseModal**: asignar curso a empleados o áreas (reusable)

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Tabla de cursos con multi-acción | 16h | — |
| Modal Asignar Curso (employees / áreas) | 20h | — |
| API: listado cursos de la empresa | — | 8h |
| API: asignación de cursos (empleados, áreas) | — | 12h |
| API: importar desde plantilla global | — | 8h |
| **Subtotal** | **36h** | **28h** |

---

### 3. Crear Curso 🟡 Media complejidad

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Formulario: Nombre, Categoría, Duración, Estado, Descripción | 14h | — |
| API: POST crear curso | — | 8h |
| **Subtotal** | **14h** | **8h** |

---

### 4. Editar Curso 🟡 Media complejidad

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Formulario pre-cargado + editor contenido básico WYSIWYG | 16h | — |
| Botón destructivo eliminar con confirmación | 6h | — |
| API: GET curso, PUT actualizar, DELETE | — | 8h |
| **Subtotal** | **22h** | **8h** |

---

### 5. CourseManage — Editor Completo 🔴 Ultra complejidad

> **La página más compleja del sistema** (~2,450 líneas en React). Sistema multi-tab completo de gestión de un curso.

#### Tab 1: Información
- Nombre, Descripción, Categoría, Estado
- Checkboxes: completado secuencial, emitir certificado
- Duración estimada total (minutos)

#### Tab 2: Contenido (Editor de Lecciones)
- Lista de lecciones con Drag & Drop para reordenar
- Formulario por lección: Título + Duración
- **Editor de bloques** por lección (Drag & Drop de bloques):
  - 5 tipos de bloque: Título | Párrafo | Imagen | Video | Archivo
  - Video: URL (YouTube/Vimeo) o subida de archivo
  - Vista previa de bloques en tiempo real
- Acciones por lección: Editar, Eliminar, Reordenar

#### Tab 3: Evaluación
- Config del examen: Duración (min), % mínimo aprobación, N° intentos
- Opciones: Mezclar preguntas, Mostrar resultados al final
- **Banco de preguntas aleatorio**: activar/desactivar, N° preguntas por intento
- Lista de preguntas: número, texto, tipo (Única/Múltiple), puntos
- Opciones con checkmarks de respuestas correctas
- Formulario crear/editar pregunta inline

#### Tab 4: Certificado
- Diseñador de certificado embebido (mismo que CertificateDesigner)

#### Tab 5: Vista Previa
- Rendering del curso completo como lo vería el empleado

| Tarea | FE Angular | BE .NET |
|---|---|---|
| **Tab: Información** — config general | 12h | — |
| **Tab: Contenido** — lista lecciones + reordenar (CDK DnD) | 20h | — |
| Editor de bloques por lección (5 tipos) | 28h | — |
| DnD para reordenar bloques dentro de lección | 12h | — |
| Upload de archivos + integración video URL | 16h | — |
| **Tab: Evaluación** — configuración examen | 16h | — |
| Lista de preguntas + formulario inline | 20h | — |
| Sistema banco aleatorio + configuración | 12h | — |
| **Tab: Certificado** — diseñador embebido | 20h | — |
| **Tab: Vista Previa** — rendering | 14h | — |
| APIs: CRUD lecciones + reordenar | — | 12h |
| APIs: CRUD bloques de contenido | — | 8h |
| API: upload de archivos (Azure Blob / S3) | — | 16h |
| APIs: CRUD preguntas, config examen | — | 12h |
| API: banco aleatorio — guardar configuración | — | 8h |
| API: guardar diseño certificado | — | 10h |
| **Subtotal** | **170h** | **66h** |

---

### 6. Seguimiento de Cursos 🟠 Alta complejidad

#### Pantallas / Componentes
- Stats: Asignados, Completados (con %), En Progreso, Promedio calificación
- **Tab: Por Empleado** — tabla con búsqueda + filtro de estado
  - Columnas: Empleado (Nombre+Área), Progreso (barra %), Estado badge, Calificación, Fecha completado, Certificado
- **Tab: Por Área** — tabla con métricas por área
  - Columnas: Área, Completados (N/Total + barra), En Progreso, No Iniciados, Promedio, Calificación Promedio, Tasa %

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Stats del curso | 8h | — |
| Tab Por Empleado — tabla con filtros y búsqueda | 16h | — |
| Barras de progreso + botón ver certificado | 8h | — |
| Tab Por Área — tabla con métricas | 14h | — |
| API: progreso por curso — empleados | — | 10h |
| API: progreso por curso — áreas | — | 10h |
| **Subtotal** | **46h** | **20h** |

---

### 7. Diseñador de Certificados 🟠 Alta complejidad

> Editor visual completo de certificados con vista previa en tiempo real.

#### Panel Izquierdo — 4 Tabs de configuración
- **Template**: 4 plantillas (Professional, Modern, Classic, Elegant) con preview de colores
- **Content**: Editores de texto (Título, Subtítulo, Cuerpo, Pie) + 10 checkboxes de info a incluir (nombre, empresa, fecha, etc.)
- **Design**: 3 color pickers (Principal, Secundario, Acento), Select fuente, Select estilo borde, Select grosor
- **Elements**: Logo empresa (upload), Firma digital (nombre+cargo+upload), Sello oficial (upload), Watermark (checkbox)

#### Panel Derecho — Vista Previa en Tiempo Real
- Renderiza el certificado con todos los elementos configurados
- Decorative corners condicionales
- Watermark con opacidad baja
- Posiciones: Firma, Sello, QR code
- Botones: Descargar PDF, Vista previa completa

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Tab Template — selector 4 plantillas con preview | 12h | — |
| Tab Content — editores texto + checkboxes info | 14h | — |
| Tab Design — color pickers, fuentes, bordes | 14h | — |
| Tab Elements — uploads logo/firma/sello/watermark | 16h | — |
| Vista previa reactiva en tiempo real | 20h | — |
| Descarga PDF (html2canvas + jsPDF) | 10h | — |
| API: guardar configuración de certificado | — | 8h |
| API: generar PDF server-side (QuestPDF) | — | 20h |
| **Subtotal** | **86h** | **28h** |

---

### 8. Gestión de Áreas 🟢 Baja complejidad

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Tabla CRUD de áreas con conteo de empleados | 14h | — |
| APIs CRUD áreas | — | 10h |
| **Subtotal** | **14h** | **10h** |

---

### 9. Gestión de Empleados 🟠 Alta complejidad

#### Pantallas / Componentes
- Alert si hay empleados sin acceso (con botón filtrar)
- Tabla con checkboxes de selección múltiple
- Columnas: Checkbox, Empleado (Avatar+Email), Área, Estado, Acceso (tiene/sin acceso), Acciones
- Por fila: "Crear Acceso" o "Reenviar Credenciales" (según estado), "Asignar Curso", Editar, Eliminar
- **Formulario nuevo empleado** con sección condicional "Crear acceso al sistema"
- **Barra de acciones masivas** (cuando hay selección múltiple):
  - Contador de seleccionados
  - Botón "Crear/Reenviar Accesos" masivo

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Tabla con checkboxes de selección múltiple | 14h | — |
| Alert de sin-acceso + filtrado rápido | 8h | — |
| Formulario nuevo empleado + sección acceso condicional | 16h | — |
| Acciones por fila (condicionales según estado acceso) | 12h | — |
| Barra de acciones masivas | 12h | — |
| Modal asignar curso desde empleado | 8h | — |
| APIs CRUD empleados + activar/desactivar acceso | — | 16h |
| API: envío de credenciales por email | — | 10h |
| Servicio email (SMTP / SendGrid) con plantilla HTML | — | 14h |
| **Subtotal** | **70h** | **40h** |

---

### 10. Roles de Empresa 🟢 Baja complejidad

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Tabla roles con icono, conteo usuarios, acciones | 12h | — |
| Formulario CRUD de roles internos | 12h | — |
| APIs CRUD roles de empresa | — | 10h |
| **Subtotal** | **24h** | **10h** |

---

### 11. Calendario 🟡 Media complejidad

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Calendario interactivo (FullCalendar / Angular Calendar) | 20h | — |
| Vista de plazos de cursos integrada | 14h | — |
| APIs: eventos del calendario, plazos de cursos | — | 14h |
| **Subtotal** | **34h** | **14h** |

---

### 12. Configuración de Empresa 🟡 Media complejidad

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Formulario config general (nombre, logo, datos) | 14h | — |
| Config de notificaciones y preferencias | 10h | — |
| APIs: GET/PUT configuración por empresa | — | 12h |
| **Subtotal** | **24h** | **12h** |

---

### 13. Usuarios de Empresa 🟡 Media complejidad

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Tabla usuarios con roles y estado | 14h | — |
| Invitar usuarios + asignar rol | 10h | — |
| APIs CRUD usuarios empresa | — | 12h |
| **Subtotal** | **24h** | **12h** |

---

## Resumen Company

| Página | Complejidad | FE | BE | Total |
|---|---|---|---|---|
| Dashboard | 🟡 Media | 22h | 12h | 34h |
| Gestión de Cursos | 🟡 Media | 36h | 28h | 64h |
| Crear Curso | 🟡 Media | 14h | 8h | 22h |
| Editar Curso | 🟡 Media | 22h | 8h | 30h |
| CourseManage ⭐ | 🔴 Ultra | 170h | 66h | 236h |
| Seguimiento Cursos | 🟠 Alta | 46h | 20h | 66h |
| Diseñador Certificados | 🟠 Alta | 86h | 28h | 114h |
| Áreas | 🟢 Baja | 14h | 10h | 24h |
| Empleados | 🟠 Alta | 70h | 40h | 110h |
| Roles | 🟢 Baja | 24h | 10h | 34h |
| Calendario | 🟡 Media | 34h | 14h | 48h |
| Configuración | 🟡 Media | 24h | 12h | 36h |
| Usuarios Empresa | 🟡 Media | 24h | 12h | 36h |
| **TOTAL** | | **586h** | **258h** | **844h** |

---

## Dependencias y orden de desarrollo

```
1. BD: Empresa, Curso, Lección, Bloque, Área, Empleado, Rol
2. APIs CRUD cursos/áreas → FE Cursos básico
3. API upload archivos → Editor de lecciones
4. APIs exámenes + banco aleatorio → Tab Evaluación
5. APIs certificado + PDF → Diseñador + descarga
6. APIs empleados + email → Gestión empleados
7. APIs tracking → Seguimiento de cursos
```

---

## 🔗 Ver también

- [[03 - Módulo SuperAdmin]]
- [[05 - Módulo Employee]]
- [[07 - Componentes y Features Especiales]]
