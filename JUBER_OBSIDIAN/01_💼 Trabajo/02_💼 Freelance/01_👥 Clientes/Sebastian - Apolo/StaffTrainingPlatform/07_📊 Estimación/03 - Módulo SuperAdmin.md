---
tags: [estimacion, superadmin, stafftraining]
fecha: 2026-06-10
modulo: SuperAdmin
fe_total: 254h
be_total: 146h
total: 400h
---

# 👑 Módulo SuperAdmin

> Gestión global de la plataforma. Administra empresas (tenants), usuarios de apoyo/soporte y cursos plantilla globales.

**Total módulo: FE 254h / BE 146h = 400h**

---

## Roles que acceden

- `superadmin` — acceso total
- `support` — acceso limitado (lectura + acciones por empresa asignada)

---

## Páginas del módulo

### 1. Dashboard SuperAdmin 🟠 Alta complejidad

> Visión global de toda la plataforma con métricas y rankings.

#### Pantallas / Componentes
- Cards de métricas: Total empresas, Activas, Empleados totales, Cursos activos
- Tabla de empresas con columnas: Nombre, RUC, Empleados, Cursos activos, Tasa completado, Estado
- Panel "Actividad Reciente" — timeline de eventos de la plataforma
- Ranking "Empresas Top por Engagement" — podio visual con barras

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Cards de métricas globales con loaders | 8h | — |
| Tabla de empresas + ranking engagement | 14h | — |
| Panel actividad reciente (timeline) | 10h | — |
| API: métricas globales (total empresas, empleados, cursos) | — | 8h |
| API: ranking empresas por engagement | — | 4h |
| API: actividad reciente (últimos N eventos) | — | 4h |
| **Subtotal** | **32h** | **16h** |

---

### 2. Listado de Empresas (Tenants) 🟡 Media complejidad

> Tabla de todas las empresas registradas en la plataforma.

#### Pantallas / Componentes
- Tabla con avatar (iniciales), Nombre, RUC, Email admin, Estado badge, Fecha creación
- Búsqueda por nombre / RUC / email
- Acciones por fila: Editar empresa, Ingresar como empresa (impersonation)

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Tabla de empresas con búsqueda y filtros | 16h | — |
| API: listado paginado de empresas | — | 8h |
| **Subtotal** | **16h** | **8h** |

---

### 3. Crear Empresa 🟡 Media complejidad

#### Pantallas / Componentes
- **Sección 1: Datos de empresa** — Nombre, RUC, Email corporativo, Teléfono, Estado, Límite empleados, Notas
- **Sección 2 (condicional):** Checkbox "Crear usuario administrador" → despliega 4 campos adicionales
- Aviso informativo sobre cambio de contraseña en primer acceso

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Formulario reactivo multi-sección + toggle condicional | 20h | — |
| API: POST crear empresa | — | 10h |
| API: POST crear usuario admin junto con empresa | — | 8h |
| **Subtotal** | **20h** | **18h** |

---

### 4. Editar Empresa 🟡 Media complejidad

#### Pantallas / Componentes
- Igual a Crear Empresa + campo "Plan" (Básico / Profesional / Empresarial)
- Botón destructivo "Eliminar Empresa" con confirmación
- Pre-carga de datos existentes

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Formulario edición + pre-carga + campo Plan | 16h | — |
| Dialog de confirmación para eliminar | 8h | — |
| API: GET empresa, PUT actualizar, DELETE eliminar | — | 12h |
| Lógica soft-delete + cascade en empleados/cursos | — | 12h |
| **Subtotal** | **24h** | **24h** |

---

### 5. Cursos Plantilla Globales 🟢 Baja complejidad

> Catálogo de cursos plantilla reutilizables por todas las empresas.

#### Pantallas / Componentes
- Tabla con: Nombre, Categoría (badge coloreado), Estado (Publicado/Borrador)
- Categorías: Seguridad (rojo), Salud (verde), Gestión (índigo), Soft Skills (púrpura)
- Acciones: Editar, Duplicar, Eliminar

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Tabla con badges de categoría y acciones | 12h | — |
| API: CRUD cursos plantilla + endpoint duplicar | — | 16h |
| **Subtotal** | **12h** | **16h** |

---

### 6. Usuarios de Apoyo / Soporte 🔴 Ultra complejidad

> Gestión de usuarios con acceso controlado a empresas específicas + sistema de permisos granular.

#### Pantallas / Componentes
- Stats: Total usuarios, Activos, Empresas cubiertas
- Tabla: Avatar+Email, Rol, Empresas asignadas (badges), Estado, Último acceso
- Acciones: Editar, Eliminar

**Formulario Crear/Editar Usuario** — MUY COMPLEJO:
- Sección 1: Nombre, Email, Contraseña (toggle mostrar/ocultar), Estado, Checkbox enviar email
- Sección 2 — Empresas y Permisos:
  - Grid de checkboxes → seleccionar qué empresas puede gestionar
  - Por cada empresa seleccionada → panel expandible con:
    - Dropdown "Plantilla de rol" (Consultor / Gestor Cursos / Gestor RRHH / Supervisor / Personalizado)
    - **Matriz 6×5** (Módulos × Acciones):
      - Módulos: Cursos, Evaluaciones, Empleados, Áreas, Avance/Reportes
      - Acciones: Ver, Crear, Editar, Eliminar, Extra
      - Checkbox individual por celda + "Seleccionar todo fila" + "Limpiar"
    - Contador de permisos activos + indicador de rol aplicado

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Listado de usuarios con stats y tabla | 16h | — |
| Formulario: sección datos personales | 14h | — |
| Grid de selección de empresas (checkboxes) | 14h | — |
| Panel de permisos por empresa (expandible) | 20h | — |
| Matriz 6×5 de permisos con checkboxes | 20h | — |
| Presets de roles + aplicar a matriz | 12h | — |
| Contador y validación de permisos | 8h | — |
| API: CRUD usuarios de apoyo | — | 12h |
| API: asignar empresas a usuario | — | 8h |
| Modelo de permisos granulares en BD | — | 16h |
| API: guardar/obtener permisos por (usuario, empresa, módulo) | — | 12h |
| **Subtotal** | **104h** | **48h** |

---

### 7. Roles de Usuarios de Apoyo 🟠 Alta complejidad

> Gestión de roles predefinidos reutilizables para usuarios de apoyo.

#### Pantallas / Componentes
- Stats: Total roles, Usuarios con rol asignado
- Grid 2 columnas con tarjetas de rol:
  - Icono coloreado, Nombre, Código, Descripción
  - Lista de permisos del rol (tags)
  - Contador de usuarios con ese rol
  - Acciones: Editar, Eliminar

**Formulario Crear/Editar Rol:**
- Nombre, Código (auto-mayúscula), Descripción
- Matriz de permisos igual a la de usuarios (sin restricción por empresa)
- Botones: "Seleccionar todo", "Limpiar"
- Checkboxes con estado indeterminado por módulo

| Tarea | FE Angular | BE .NET |
|---|---|---|
| Grid de tarjetas de roles con stats | 14h | — |
| Formulario: nombre, código auto-mayúscula | 12h | — |
| Matriz permisos global (igual que usuarios) | 20h | — |
| Checkbox indeterminado por módulo | 8h | — |
| API: CRUD roles de apoyo | — | 10h |
| API: guardar permisos por rol | — | 10h |
| **Subtotal** | **54h** | **20h** |

---

## Resumen SuperAdmin

| Página | Complejidad | FE | BE | Total |
|---|---|---|---|---|
| Dashboard | 🟠 Alta | 32h | 16h | 48h |
| Listado Empresas | 🟡 Media | 16h | 8h | 24h |
| Crear Empresa | 🟡 Media | 20h | 18h | 38h |
| Editar Empresa | 🟡 Media | 24h | 24h | 48h |
| Cursos Plantilla | 🟢 Baja | 12h | 16h | 28h |
| Usuarios Apoyo | 🔴 Ultra | 104h | 48h | 152h |
| Roles Apoyo | 🟠 Alta | 54h | 20h | 74h |
| **TOTAL** | | **262h** | **150h** | **412h** |

> *Nota: incluye 8h adicionales FE vs resumen por desglose más fino.*

---

## Dependencias y orden de desarrollo

```
1. Setup + Auth (.NET)
2. Modelo BD: Empresa, Usuario, Rol, Permisos
3. APIs CRUD empresas → Frontend Empresas
4. APIs CRUD usuarios SA → Frontend Usuarios SA
5. Modelo permisos granular → Matriz FE
6. Roles SA → reutiliza el componente de matriz
```

---

## 🔗 Ver también

- [[01 - Resumen Ejecutivo]]
- [[04 - Módulo Company]]
- [[09 - Inventario del Proyecto]]
