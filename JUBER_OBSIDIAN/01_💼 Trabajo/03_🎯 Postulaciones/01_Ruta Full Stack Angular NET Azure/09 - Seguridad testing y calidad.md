---
tags:
  - Postulaciones
  - Seguridad
  - Testing
  - Calidad
Creado: 2026-08-30
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Seguridad, testing y calidad

## Seguridad que sí debe dominar un Full Stack

### Identidad y acceso

- Diferenciar autenticación y autorización.
- OAuth 2.0 y OpenID Connect a nivel de flujos y responsabilidades.
- Access tokens cortos y estrategia segura de refresh.
- Claims, roles, policies y autorización por recurso.
- Microsoft Entra ID cuando el entorno lo requiera.

### Aplicación web

- XSS, CSRF, CORS y clickjacking.
- Validación en cliente para UX y en servidor como autoridad.
- SQL injection y parametrización.
- Mass assignment/over-posting.
- Rate limiting, límites de payload y abuso.
- Secrets fuera del código y del frontend.
- HTTPS, cabeceras y cookies seguras cuando corresponda.

Repasar: [[Seguridad Auth JWT dotNET]], [[Auth JWT Refresh Tokens RBAC]], [[06 - OWASP para APIs NET]].

## Pirámide práctica de pruebas

| Nivel | Qué protege | Herramienta o enfoque |
|---|---|---|
| Unit | reglas puras y casos de uso | xUnit / tests TypeScript |
| Component | UI y comportamiento aislado | Angular testing |
| Integration | API, middleware, auth y DB | WebApplicationFactory + DB real |
| Contract | compatibilidad frontend/API | OpenAPI y validación de contratos |
| E2E | pocos recorridos críticos | Playwright o equivalente |

No buscar 100 % de cobertura. Priorizar autenticación, permisos, dinero, mutaciones, cálculos, concurrencia y manejo de errores.

La guía oficial de ASP.NET Core usa `WebApplicationFactory` para probar la aplicación y permite sustituir autenticación o persistencia según el escenario: [integration tests en ASP.NET Core](https://learn.microsoft.com/aspnet/core/test/integration-tests).

## Quality gate del repositorio

- [ ] Compila desde cero.
- [ ] Lint/format pasan.
- [ ] No hay warnings críticos ignorados.
- [ ] Tests unitarios y de integración pasan.
- [ ] Dependencias vulnerables se revisan.
- [ ] Secret scanning activo.
- [ ] Imágenes se escanean.
- [ ] Pull request requiere revisión.
- [ ] README explica ejecución y decisiones.

## Historia que debes poder contar

Preparar un ejemplo de cada tipo:

- bug evitado por una prueba;
- vulnerabilidad o permiso mal planteado;
- refactor que redujo complejidad;
- fallo de integración diagnosticado;
- decisión de no automatizar o no abstraer demasiado.

## Criterio de salida

- [ ] Puedo diseñar una estrategia de pruebas por riesgo.
- [ ] Puedo explicar dónde termina la responsabilidad de Angular.
- [ ] Autorizo en backend, no solo oculto botones.
- [ ] Sé probar una API con base de datos y autenticación.
- [ ] Puedo revisar un PR buscando seguridad, corrección y operabilidad.
