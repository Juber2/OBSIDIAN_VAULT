---
tags: [estimacion, employee, stafftraining]
fecha: 2026-06-10
modulo: Employee
fe_total: 272h
be_total: 114h
total: 386h
---

# 👤 Módulo Employee

> Experiencia de aprendizaje para el empleado. Visualiza y completa cursos, toma exámenes y descarga certificados.

**Total módulo: FE 272h / BE 114h = 386h**

---

## Rol que accede

- `employee` — empleado de la empresa

---

## Páginas del módulo

### 1. Dashboard Empleado 🟡 Media complejidad

#### Pantallas / Componentes
- Saludo personalizado con nombre del empleado
- Stats (4 cards): Cursos en progreso, Completados, Horas de capacitación, Certificados
- Panel "Mis Cursos Activos": cards con barra de progreso, siguiente lección, fecha límite
- Sidebar derecho:
  - "Próximas Evaluaciones": cards con título, fecha, hora, duración
  - "Logros Recientes": título, descripción, emoji icono, fecha

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Saludo + stats cards | 10h | — |
| Panel cursos activos con progreso y fechas | 14h | — |
| Sidebar: próximas evaluaciones + logros | 8h | — |
| API: datos del dashboard del empleado autenticado | — | 14h |
| **Subtotal** | **32h** | **14h** |

---

### 2. Mis Cursos 🟠 Alta complejidad

#### Pantallas / Componentes
- Búsqueda global de cursos
- **Tab: En Progreso** — grid de cards (3 cols)
  - Thumbnail degradado + badge categoría
  - Título, descripción
  - Módulos completados/totales + barra de progreso %
  - Fecha límite + siguiente lección
  - Botón "Continuar Curso"
- **Tab: Completados** — cards con checkmark en thumbnail
  - Calificación obtenida (score)
  - Botón "Ver Certificado"
- **Tab: Disponibles** — cards sin progreso
  - Duración, Módulos, Nivel (badge)
  - Botón "Inscribirse al Curso"

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Búsqueda global de cursos | 8h | — |
| Tab En Progreso — grid de cards con progreso | 14h | — |
| Tab Completados — cards con calificación | 12h | — |
| Tab Disponibles — cards con inscripción | 12h | — |
| API: cursos del empleado por estado | — | 10h |
| API: inscribirse a curso disponible | — | 4h |
| **Subtotal** | **46h** | **14h** |

---

### 3. Vista de Curso 🔴 Ultra complejidad

> Sistema completo de reproducción de curso con lecciones, video, evaluación y certificado.

#### Tab: Contenido (Grid 3 columnas)
**Sidebar — Lista de lecciones:**
- Lección actual: resaltada con degradado indigo-purple
- Lecciones completadas: checkmark verde
- Lecciones pendientes: círculo vacío
- Cada ítem: Icono + Título + Duración

**Main — Contenido de lección:**
- Título + Badge estado + Duración
- Video player (iframe YouTube/Vimeo, aspecto 16:9)
- Descripción de la lección
- Sección "Recursos Descargables": icono PDF, nombre, tamaño, botón descargar
- Navegación: "Lección Anterior" / "Marcar como completada" / "Siguiente"

**Panel lateral — Progreso general:**
- Card con degradado mostrando % total

#### Tab: Evaluación
- Card con ícono Award + Nombre de la evaluación
- Stats: N° preguntas, % aprobación, Duración
- Intentos disponibles / usados
- Si bloqueado (progreso < 100%): Card rojo con ícono Lock
- Si desbloqueado: Botón "Comenzar Evaluación"
- Card de instrucciones colapsable

#### Tab: Certificado
- Si NO obtenido: Card con requisitos y checkmarks dinámicos según progreso
- Si obtenido: Card verde + vista previa del certificado + botones Descargar / Compartir

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Sidebar lecciones (completada/activa/pendiente) | 16h | — |
| Video player (YouTube/Vimeo embed) | 14h | — |
| Recursos descargables | 8h | — |
| Navegación entre lecciones + marcar completada | 12h | — |
| Tab Evaluación — info + bloqueo condicional | 14h | — |
| Tab Certificado — condicional según estado | 12h | — |
| API: obtener lecciones del curso con estado | — | 8h |
| API: marcar lección como completada | — | 4h |
| API: estado de progreso del empleado en el curso | — | 6h |
| **Subtotal** | **76h** | **18h** |

---

### 4. Sistema de Exámenes 🟠 Alta complejidad

#### ExamView — Vista del examen
- Pantalla inicio: nombre, instrucciones, config (tiempo, preguntas, intentos)
- Timer visible (countdown)
- Navegación entre preguntas (barra lateral o paginación)
- Tipos de pregunta: Opción única (radio), Opción múltiple (checkboxes)
- Indicador de preguntas respondidas/pendientes
- Botón "Enviar examen" con confirmación

#### ExamResult — Resultado del examen
- Score obtenido vs score mínimo
- Badge: "Aprobado ✓" o "Reprobado ✗"
- Tiempo utilizado / N° intento
- Resumen de respuestas (si está configurado)
- Botón: "Ver Certificado" (si aprobó) o "Reintentar" (si falló)

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Pantalla inicio del examen | 10h | — |
| Vista examen: timer + navegación preguntas | 24h | — |
| Tipos pregunta: radio / checkbox | 8h | — |
| Pantalla resultado: score + resumen | 16h | — |
| API: iniciar intento de examen | — | 8h |
| API: obtener preguntas (respetando banco aleatorio) | — | 10h |
| API: enviar respuestas + calificar automáticamente | — | 12h |
| API: registrar intento y resultado | — | 6h |
| Lógica banco aleatorio en BE | — | 4h |
| **Subtotal** | **58h** | **40h** |

---

### 5. Certificados 🟡 Media complejidad

#### CertificateView — Ver certificado individual
- Renderizado del certificado con diseño configurado por la empresa
- Datos: nombre empleado, curso, fecha, calificación, firma, sello, QR

#### Listado Mis Certificados
- Cards o tabla de todos los certificados obtenidos
- Filtros por fecha, curso
- Botón descargar PDF por certificado

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Listado de certificados obtenidos | 12h | — |
| Vista individual del certificado (renderizado) | 16h | — |
| Descarga PDF del certificado | 8h | — |
| API: listado de certificados del empleado | — | 6h |
| API: datos del certificado + generar PDF | — | 8h |
| **Subtotal** | **36h** | **14h** |

---

### 6. Mi Progreso & Evaluaciones 🟡 Media complejidad

#### Mi Progreso
- Dashboard de progreso general
- Gráficos: cursos completados por mes, horas de capacitación
- Historial de actividades

#### Mis Evaluaciones
- Tabla de evaluaciones: Curso, Fecha, Score, Estado (Aprobado/Reprobado), Intento
- Filtros por estado y fecha

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Dashboard progreso con gráficos (recharts/ngx-charts) | 18h | — |
| Listado de evaluaciones con filtros | 14h | — |
| API: historial de progreso del empleado | — | 8h |
| API: historial de evaluaciones del empleado | — | 6h |
| **Subtotal** | **32h** | **14h** |

---

## Resumen Employee

| Página | Complejidad | FE | BE | Total |
|---|---|---|---|---|
| Dashboard | 🟡 Media | 32h | 14h | 46h |
| Mis Cursos | 🟠 Alta | 46h | 14h | 60h |
| Vista de Curso | 🔴 Ultra | 76h | 18h | 94h |
| Sistema de Exámenes | 🟠 Alta | 58h | 40h | 98h |
| Certificados | 🟡 Media | 36h | 14h | 50h |
| Progreso & Evaluaciones | 🟡 Media | 32h | 14h | 46h |
| **TOTAL** | | **280h** | **114h** | **394h** |

---

## Dependencias y orden de desarrollo

```
1. API: progreso del empleado (base para todo)
2. API: lecciones + marcar completada → Vista de Curso
3. API: examen + calificación → Sistema Exámenes
4. API: certificados + PDF → Certificados
5. API: historial → Mi Progreso
```

---

## 🔗 Ver también

- [[04 - Módulo Company]]
- [[06 - Setup e Infraestructura]]
- [[09 - Inventario del Proyecto]]
