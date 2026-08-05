---
tags:
  - adr
  - decisiones
  - cursos
  - evaluaciones
  - versionado
  - staff-training-platform
Fecha: 2026-06-22
Estado: Activo
---

# ADR-004 — Versionado de Cursos y Evaluaciones

## Problema

Una persona inicia un curso mientras el admin lo edita en paralelo → inconsistencia entre lo que la persona ve/cursó y lo que realmente existe ahora en el sistema.

## Decisión raíz: versionado de contenido (no locking)

En vez de bloquear la edición mientras hay gente cursando, cada publicación de cambios crea una nueva versión inmutable. Quien ya está inscrito queda fijo a la versión con la que empezó; el admin edita libremente sin afectar a nadie en curso.

---

## Estructura de tablas

### `courses`
- Contenedor estable: título, descripción, `is_active`.
- **No** guarda `passing_score` ni `duration_hours` directamente — esos viven en la versión.

### `course_versions`
Unidad atómica de versionado.

| Campo | Detalle |
|---|---|
| `course_id` | FK a `courses` |
| `version_number` | Incremental |
| `status` | `draft` / `published` / `archived` — historial de qué pasó con esa fila, no selector de "la actual" |
| `exam_template_id` | Nullable — referencia copy-on-write al examen vigente para esa versión |
| `is_current` | Boolean — identifica la versión activa para nuevas inscripciones |

> `is_current` reemplaza la idea descartada de `courses.current_version_id`, que generaba una FK circular entre ambas tablas.

### `course_modules` → `module_contents` → `module_content_files`
- Cuelgan de `course_version_id` (ya **no** de `course_id`).

### `person_courses`
- Cuelga de `course_version_id` únicamente.
- Se eliminó `course_id` directo — normalización completa (Opción 2).
- Ya sin `attempts` / `final_score` — esos campos viven en `exam_instance_attempts`.

### `certificates`
- Apunta a `person_course_id` (no a `person_id + course_version_id` sueltos).
- Evita ambigüedad y permite derivar `company_id` sin riesgo de inconsistencia.

### `certificate_templates`
- Apunta a `course_company_id` (vía `course_companies`), **no** a `course_id`.
- No se versiona por curso — es un molde reutilizable.
- El certificado ya emitido es su propio snapshot inmutable (`certificate_url`).

---

## Evaluaciones — capa hermana, no anidada

### `exam_templates`
- Apunta a `course_id` (no a `course_version_id`).
- Inmutable una vez publicado — "editar" crea una fila nueva.
- Tiene su propio `version_number` / `status`.
- El vínculo real con una versión específica de curso vive en `course_versions.exam_template_id` (copy-on-write) → nunca se evalúa a alguien con contenido que no estudió.

### `exam_instances` → `exam_instance_questions` → `exam_instance_question_options`
- Adelgazadas — ya **no** copian texto/tipo/puntos/opciones del template.
- Solo guardan `order_index` (resultado del shuffle de preguntas y opciones) por persona/instancia.

### `exam_instance_attempts` → `exam_instance_question_answers`
- Sin cambios — son intrínsecamente datos por intento.

---

## Pendientes resueltos (2026-06-22)

### Momento de fijación de `exam_template_id` y `course_version_id`
Se fijan en `created_at` de `person_courses` — es decir, en el momento de **asignación**, no de inicio real (`start_date`).

El sistema no depende de cuándo el usuario decide empezar a cursar, ya que eso está fuera de su control. El desfase entre asignación e inicio real es un **riesgo de negocio aceptado y documentado**, no un bug pendiente: si el contenido o el examen cambian en ese lapso, la persona cursa con lo que se le asignó originalmente.

### Soft-delete en `exam_template_questions` / `exam_template_question_options`
No se necesita soft-delete adicional. La inmutabilidad ya está garantizada por el versionado a nivel de `exam_templates` — editar crea una fila nueva con `version_number` incrementado. **Cerrado sin cambios al esquema.**

---

## Consecuencias

- El admin puede editar cursos en cualquier momento sin afectar a personas en curso.
- Nuevas inscripciones siempre toman la versión marcada como `is_current`.
- La historia de evaluaciones es trazable por versión de curso.
- Evita FK circular entre `courses` y `course_versions`.
