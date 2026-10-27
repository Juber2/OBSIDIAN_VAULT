---
tags:
  - Postulaciones
  - Portfolio
  - FullStack
Creado: 2026-08-30
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Proyecto de portafolio demostrable

## Decisión

No comenzar otro proyecto enorme. Convertir **StaffTrainingPlatform** en la evidencia principal y completar una vertical slice de calidad productiva.

## Alcance demostrable

### Producto

- Login y autorización por roles/policies.
- Gestión de una entidad principal con listado, filtros, paginación, creación y edición.
- Validación y manejo consistente de errores.
- Subida de un archivo a Blob Storage.
- Un proceso asíncrono, por ejemplo notificación o generación de reporte.

### Angular

- Feature architecture.
- Typed forms.
- Interceptor y manejo de sesión.
- Signals/RxJS con una decisión documentada.
- Estados loading/error/empty.
- Tests del flujo crítico.

### .NET

- API REST y OpenAPI.
- EF Core con queries eficientes.
- Authorization policies.
- Integration tests con base real.
- Background worker y operación idempotente.
- Health checks y ProblemDetails.

### Plataforma

- Dockerfiles multi-stage.
- Compose para entorno local.
- CI/CD.
- Deploy en Azure.
- Key Vault + managed identity.
- OpenTelemetry + Application Insights.
- Dashboard y una alerta.

## Base de datos

Mantener SQL Server/Azure SQL en el proyecto principal si ya está integrado. Crear un laboratorio separado y pequeño con PostgreSQL para demostrar transferencia. No mezclar dos motores en el mismo producto sin una razón de negocio.

MongoDB no se agrega salvo que aparezca un agregado realmente documental. Los logs van a la plataforma de observabilidad.

## Repositorio profesional

El README debe incluir:

- problema que resuelve;
- arquitectura con diagrama;
- stack y decisiones;
- requisitos y ejecución local;
- variables sin secretos;
- estrategia de pruebas;
- pipeline y despliegue;
- observabilidad;
- trade-offs y siguientes pasos;
- capturas o demo breve.

## ADR mínimos

- [ ] App Service vs Container Apps.
- [ ] SQL Server vs PostgreSQL.
- [ ] Modular monolith vs microservicios.
- [ ] Signals/RxJS/estado global.
- [ ] Procesamiento síncrono vs Service Bus.
- [ ] Estrategia de autenticación y secretos.

## Demo de cinco minutos

1. Explicar problema y usuario.
2. Ejecutar el flujo principal.
3. Mostrar un test relevante.
4. Mostrar pipeline y despliegue.
5. Provocar o localizar un error en observabilidad.

## Definición de terminado

- [ ] Una persona externa puede ejecutar el repositorio.
- [ ] La URL desplegada funciona.
- [ ] Ningún secreto está versionado.
- [ ] El pipeline es verde.
- [ ] Los flujos principales tienen pruebas.
- [ ] Existe telemetría correlacionada.
- [ ] Puedes explicar tres decisiones y dos limitaciones.

