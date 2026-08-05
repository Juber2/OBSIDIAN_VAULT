---
tags: [estimacion, supuestos, metodologia]
fecha: 2026-06-10
---

# ⚙️ Supuestos y Metodología

## Parámetros base del cálculo

| Parámetro | Valor |
|---|---|
| Jornada laboral | 8 horas/día |
| Días laborables/semana | 5 días |
| Duración de sprint | 2 semanas = 80h por dev |
| Buffer de contingencia | +20% sobre estimación base |
| Nivel del equipo asumido | Senior (3+ años en el stack) |

---

## Qué incluye la estimación

- ✅ Análisis y diseño de cada pantalla/componente
- ✅ Desarrollo de componentes Angular (template, lógica, estilos)
- ✅ Servicios Angular (HTTP, state, reactive forms)
- ✅ Guards, interceptors, resolvers
- ✅ Endpoints REST .NET (controller, service, repository)
- ✅ Modelos de dominio y entidades EF Core
- ✅ DTOs + validaciones (FluentValidation)
- ✅ Migraciones de base de datos
- ✅ Unit tests (componentes críticos FE, servicios BE)
- ✅ E2E tests de flujos principales

## Qué NO incluye

- ❌ Infraestructura cloud (Azure, AWS) — costos de servidores
- ❌ Licencias de software (librerías premium, servicios email, storage)
- ❌ Gestión de proyecto (PM, reuniones, dailies)
- ❌ Diseño UX/UI adicional (se usa el diseño React/Figma existente)
- ❌ Formación del equipo en el stack
- ❌ Mantenimiento post-lanzamiento

---

## Stack tecnológico asumido

### Frontend — Angular
| Tecnología | Uso |
|---|---|
| Angular 17+ | Framework principal |
| Angular Material / PrimeNG | Componentes UI |
| Angular CDK | Drag & Drop |
| RxJS | Programación reactiva |
| Angular Reactive Forms | Formularios complejos |
| NgRx o Services+Signals | State management |
| Jasmine / Jest | Unit testing |
| Playwright o Cypress | E2E testing |
| jsPDF + html2canvas | Generación de PDF |

### Backend — .NET
| Tecnología | Uso |
|---|---|
| .NET 8 | Runtime |
| ASP.NET Core Web API | API REST |
| Entity Framework Core 8 | ORM |
| SQL Server / PostgreSQL | Base de datos |
| JWT Bearer | Autenticación |
| FluentValidation | Validaciones |
| AutoMapper | DTO mapping |
| xUnit | Testing |
| Swagger / OpenAPI | Documentación API |
| SignalR | Notificaciones real-time |
| QuestPDF o IronPDF | Generación PDF server-side |

---

## Metodología de estimación

### Escala de complejidad usada

| Complejidad | Criterios | Rango FE típico |
|---|---|---|
| 🟢 Baja | Tabla simple, formulario básico, listado estático | 8–16h |
| 🟡 Media | Tabla con filtros, formulario multi-sección, tabs | 16–30h |
| 🟠 Alta | Múltiples tabs, lógica condicional compleja, modales | 30–60h |
| 🔴 Ultra | Multi-tab con editor dinámico, drag-drop, real-time | 60–170h |

### Desglose típico por página (FE)

```
Total página = Componente base + Servicio HTTP + Formulario + Integración
              + Validaciones + Responsive + Estado loading/error
```

### Desglose típico por endpoint (BE)

```
Total endpoint = Entity/Migration + Repository + Service + Controller
               + DTOs + Validación + Autorización por rol
```

---

## Riesgos identificados

| Riesgo | Impacto | Probabilidad | Mitigación |
|---|---|---|---|
| CourseManage muy complejo | Alto | Alta | Asignar 2 devs FE ese sprint |
| Diseñador certificados cross-browser | Medio | Media | Validar con html2canvas temprano |
| Modelo de permisos granular (5×6 matriz) | Alto | Media | Diseñar BD de permisos en sprint 1 |
| Upload de archivos + storage | Medio | Baja | Definir proveedor (Azure Blob) en sprint 1 |
| Integración video (YouTube/Vimeo embed) | Bajo | Baja | Usar ngx-youtube-player o similar |
| Multi-tenancy en .NET | Alto | Media | Implementar middleware tenant desde inicio |

---

## Fuente del análisis

La estimación se basa en un análisis exhaustivo del código React/Figma del proyecto actual:
- **51+ archivos `.tsx`** analizados
- **~10,000+ líneas de código** revisadas
- **35+ páginas** identificadas (19 completamente implementadas)
- **3 módulos** principales + autenticación

> Ver inventario completo → [[09 - Inventario del Proyecto]]
