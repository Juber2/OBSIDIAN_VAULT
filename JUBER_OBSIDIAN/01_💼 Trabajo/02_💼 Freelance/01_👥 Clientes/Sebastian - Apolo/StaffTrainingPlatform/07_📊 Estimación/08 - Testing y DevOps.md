---
tags: [estimacion, testing, devops, qa]
fecha: 2026-06-10
fe_total: 115h
be_total: 79h
total: 194h
---

# 🧪 Testing y DevOps

**Total: FE 115h / BE 79h = 194h**

---

## Testing Frontend (Angular)

### Unit Tests — Jasmine / Jest

Se testean los componentes y servicios **más críticos** (~30% de cobertura).

| Área | Qué se testea | Horas |
|---|---|---|
| Auth Service | Login, logout, token decode, guards | 6h |
| HTTP Interceptor | Headers, manejo de 401, retry | 4h |
| Tabla genérica | Búsqueda, paginación, selección | 6h |
| Formulario Crear Empresa | Validaciones, sección condicional | 4h |
| Matriz de permisos | Toggle checkboxes, presets, counter | 8h |
| CourseManage — Editor bloques | Agregar/eliminar bloques, tipos | 8h |
| Sistema Exámenes — Timer | Countdown, auto-envío al tiempo | 6h |
| Diseñador Certificados | Preview reactiva, persistencia | 6h |
| **Subtotal Unit Tests FE** | | **48h** |

### E2E Tests — Playwright

Se cubren los **flujos principales** del negocio.

| Flujo | Descripción | Horas |
|---|---|---|
| Login por rol | SuperAdmin, Company, Employee — login y redirect correcto | 4h |
| Crear empresa | Flujo completo con usuario admin | 4h |
| Crear curso + lecciones | Crear curso → agregar lección → agregar bloque video | 6h |
| Asignar curso | Asignar a empleado + verificar en módulo employee | 4h |
| Completar lección | Employee navega y marca lección como completada | 4h |
| Tomar examen | Employee inicia, responde y recibe resultado | 6h |
| Descargar certificado | Employee aprueba → descarga PDF | 4h |
| Crear usuario apoyo + permisos | SuperAdmin crea usuario con matriz de permisos | 6h |
| **Subtotal E2E FE** | | **38h** |

### Otros

| Tarea | Horas |
|---|---|
| Setup de testing (Jest config, Playwright config) | 6h |
| Fixtures y mocks base | 6h |
| CI: ejecutar tests en cada PR | 4h |
| Corregir tests rotos después de refactors | 13h |
| **Subtotal Otros FE** | | **29h** |

**Total Testing FE: 48 + 38 + 29 = 115h**

---

## Testing Backend (.NET)

### Unit Tests — xUnit

| Área | Qué se testea | Horas |
|---|---|---|
| Auth Service | Generación JWT, validación, refresh | 6h |
| Empresa Service | CRUD, soft-delete, cascade | 6h |
| Permisos Service | Lógica de permisos granulares | 8h |
| Curso Service | CRUD, importar plantilla, estado | 6h |
| Examen Service | Calificación automática, banco aleatorio | 10h |
| Progreso Service | Marcar completada, calcular % | 6h |
| Certificado Service | Generar tras aprobar, datos correctos | 6h |
| **Subtotal Unit Tests BE** | | **48h** |

### Integration Tests

| Área | Descripción | Horas |
|---|---|---|
| Auth endpoints | POST /auth/login, /refresh, /logout | 4h |
| Empresa endpoints | CRUD completo con DB de test | 6h |
| Examen endpoints | Iniciar intento, enviar respuestas, calificar | 8h |
| **Subtotal Integration BE** | | **18h** |

### Otros

| Tarea | Horas |
|---|---|
| Setup xUnit + InMemory DB / Testcontainers | 6h |
| Builders y factories para test data | 4h |
| **Subtotal Otros BE** | | **13h** |

**Total Testing BE: 48 + 18 + 13 = 79h**

---

## DevOps & Despliegue

### CI/CD Pipeline

Se asume **GitHub Actions** o **Azure DevOps**.

```yaml
# Pipeline por rama feature/*
on: [push, pull_request]
jobs:
  frontend:
    - Install dependencies
    - Lint (ESLint)
    - Unit tests (Jest)
    - Build (ng build --prod)
  backend:
    - Restore packages
    - Build (.NET)
    - Unit tests (xUnit)
    - Integration tests

# Pipeline main/staging
  deploy-staging:
    - Build Docker images
    - Push to container registry
    - Deploy to staging environment
    - Run smoke tests

# Pipeline main → prod (manual trigger)
  deploy-prod:
    - Approval gate
    - Run DB migrations
    - Deploy API
    - Deploy Angular SPA
    - Health check
```

| Tarea | Horas |
|---|---|
| Pipeline CI para FE (lint + test + build) | 6h |
| Pipeline CI para BE (build + test) | 6h |
| Pipeline CD a staging (automático) | 8h |
| Pipeline CD a producción (con approval gate) | 6h |
| **Subtotal CI/CD** | | **26h** |

---

### Configuración de Entornos

| Tarea | Horas |
|---|---|
| Docker Compose para desarrollo local (BE + BD) | 6h |
| Dockerfile para API .NET | 4h |
| Dockerfile para Angular (nginx) | 4h |
| Variables de entorno + Azure Key Vault / GitHub Secrets | 4h |
| **Subtotal Entornos** | | **18h** |

---

### Base de Datos Producción

| Tarea | Horas |
|---|---|
| Script migraciones automáticas al deploy | 4h |
| Seed data de producción (roles base, plantillas) | 4h |
| Backup strategy + documentación | 2h |
| **Subtotal BD** | | **10h** |

---

### Documentación

| Tarea | Horas |
|---|---|
| Swagger refinado con ejemplos y descripciones | 6h |
| README de setup local (FE + BE) | 4h |
| Guía de despliegue | 4h |
| **Subtotal Docs** | | **14h** |

---

## Resumen Testing & DevOps

| Área | FE | BE | Total |
|---|---|---|---|
| Unit Tests | 48h | 48h | 96h |
| E2E / Integration Tests | 38h | 18h | 56h |
| Setup testing | 29h | 13h | 42h |
| CI/CD | 13h | 13h | 26h |
| Entornos + Docker | 11h | 7h | 18h |
| BD producción | — | 10h | 10h |
| Documentación | 7h | 7h | 14h |
| **TOTAL** | **146h** | **116h** | **262h** |

> *Nota: DevOps se divide entre FE y BE igualmente. Los totales en el Resumen Ejecutivo distribuyen ~46h de DevOps compartido.*

---

## 🔗 Ver también

- [[01 - Resumen Ejecutivo]]
- [[02 - Supuestos y Metodología]]
- [[06 - Setup e Infraestructura]]
