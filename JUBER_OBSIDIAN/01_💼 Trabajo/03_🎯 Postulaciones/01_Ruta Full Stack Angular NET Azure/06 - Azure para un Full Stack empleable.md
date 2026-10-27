---
tags:
  - Postulaciones
  - Azure
  - Cloud
Creado: 2026-08-30
Relacionado:
  - Azure
  - RUTA FULL STACK ANGULAR NET AZURE
---

# Azure para un Full Stack empleable

## Objetivo

No memorizar cien servicios. Poder desplegar una solución segura, observarla y explicar por qué elegiste cada componente.

## Arquitectura mínima de portafolio

```text
Angular ──► Static Web Apps o App Service
                  │ HTTPS
                  ▼
           ASP.NET Core API
            App Service o
           Container Apps
             │     │
             │     ├── Key Vault + Managed Identity
             │     ├── Blob Storage
             │     └── Application Insights
             ▼
       Azure SQL Database
```

## P0 — Aprender y demostrar

### Hosting

- App Service para una API/web PaaS sencilla.
- Container Apps para containers, revisiones y escalado sin administrar Kubernetes.
- Static Web Apps o hosting estático para Angular según el caso.

### Datos y configuración

- Azure SQL Database.
- Blob Storage para archivos.
- Key Vault para secretos.
- Managed Identity para evitar credenciales embebidas.
- App Configuration como siguiente paso cuando la configuración crece.

### Operación

- Azure Monitor y Application Insights.
- Log Analytics y consultas KQL básicas.
- Alerts, availability checks y dashboards.

La documentación oficial recomienda managed identity para que aplicaciones .NET alojadas en Azure accedan a otros recursos sin administrar credenciales: [managed identity para .NET](https://learn.microsoft.com/dotnet/azure/sdk/authentication/system-assigned-managed-identity).

## P1 — Añadir después del despliegue base

- Azure Service Bus para procesos asíncronos.
- Azure Managed Redis para caché distribuida.
- Azure Container Registry.
- Functions para tareas event-driven pequeñas.
- API Management cuando haya una necesidad real de gateway, políticas, cuotas o publicación de APIs.
- Bicep o Terraform para reproducir infraestructura.

## P2 — Solo por necesidad

- AKS.
- Application Gateway y Front Door avanzados.
- Redes privadas complejas.
- Data Factory.
- Arquitectura multirregión.

AKS suma valor, pero no antes de poder desplegar y operar correctamente en App Service o Container Apps.

## Laboratorio de seis pasos

- [ ] Desplegar API y frontend.
- [ ] Conectar Azure SQL mediante configuración segura.
- [ ] Activar managed identity y Key Vault.
- [ ] Subir archivos a Blob Storage.
- [ ] Instrumentar con Application Insights/OpenTelemetry.
- [ ] Automatizar el despliegue con CI/CD.

## Preguntas de entrevista

- ¿App Service, Container Apps o AKS?
- ¿Dónde guardas secretos y cómo rota una credencial?
- ¿Cómo diagnosticas un aumento de errores 500?
- ¿Cómo despliegas sin indisponibilidad?
- ¿Qué recurso escala y cuál puede convertirse en cuello de botella?
- ¿Cómo controlarías costo, seguridad y observabilidad?

Material existente: [[01 - PaaS vs IaaS vs Cloud Native]], [[03 - App Service y Azure Functions]], [[06 - SRE APM y IaC]], [[Azure App Service SQL Key Vault Insights]].

