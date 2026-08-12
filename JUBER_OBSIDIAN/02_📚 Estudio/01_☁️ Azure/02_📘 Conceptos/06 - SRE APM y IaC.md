---
tags:
  - Azure
  - SRE
  - DevOps
Creado: 2026-08-12
Relacionado:
  - Azure
---

# SRE, APM, IaaS e IaC (deseables del puesto)

## SRE (Site Reliability Engineering)

Disciplina (originada en Google) que aplica prácticas de ingeniería de software a la operación de sistemas — la idea central: la confiabilidad se **mide y se diseña**, no se espera que "simplemente pase".

Conceptos clave que suelen preguntarse:

| Concepto | Qué es |
|---|---|
| **SLI** (Service Level Indicator) | una métrica medible (ej. % de requests exitosas, latencia p99) |
| **SLO** (Service Level Objective) | el objetivo interno sobre ese SLI (ej. "99.9% de requests exitosas") |
| **SLA** (Service Level Agreement) | el compromiso contractual con el cliente, con consecuencias si no se cumple |
| **Error Budget** | cuánto "margen de falla" permite el SLO (si el SLO es 99.9%, el error budget es 0.1%) — se usa para decidir si tomar riesgos (deploys, features) o frenar y estabilizar |

```
SLA (contrato con el cliente) ⊇ SLO (objetivo interno, más estricto) ⊇ SLI (la métrica real)
```

**Ejemplo práctico:** si el error budget del mes ya se consumió por incidentes, el equipo prioriza estabilidad sobre nuevas features hasta el siguiente período — esta es la disciplina que diferencia SRE de "operaciones tradicionales".

## APM (Application Performance Monitoring)

Herramientas/prácticas para observar el comportamiento de una app en producción en tiempo real: tiempos de respuesta, errores, trazas de requests entre microservicios.

Los "tres pilares de observabilidad":

| Pilar | Qué responde | Herramientas típicas |
|---|---|---|
| **Logs** | qué pasó exactamente en un momento dado | Azure Monitor, ELK, Loki |
| **Metrics** | cómo se comporta el sistema en agregado (CPU, latencia, throughput) | Prometheus, Azure Monitor Metrics |
| **Traces** | el recorrido completo de una request entre varios microservicios | Application Insights, Jaeger, Zipkin |

## Application Insights (el APM nativo de Azure)

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>applicationinsights-spring-boot-starter</artifactId>
</dependency>
```

```properties
azure.application-insights.instrumentation-key=${APPINSIGHTS_KEY}
```

Con esto, sin instrumentar cada endpoint a mano, Application Insights captura automáticamente: tiempos de respuesta por endpoint, excepciones no manejadas, dependencias externas (llamadas a BD, otras APIs), y arma un **mapa de dependencias** entre microservicios — muy relevante en un contexto donde ya se pidió [[03 - Services y Networking|Kubernetes con varios microservicios]].

## Health checks y Actuator (la base de la observabilidad en Spring)

```
GET /actuator/health → {"status": "UP"}
```

Esto ya lo tenés cubierto en [[07 - Actuator]] — es la pieza que un APM/Kubernetes (liveness/readiness probes) consume para saber si la app está sana.

## IaaS vs IaC (no confundir)

- **IaaS** (Infrastructure as a Service): el modelo de servicio cloud donde alquilás VMs/redes/storage crudo (ya visto en [[01 - PaaS vs IaaS vs Cloud Native]]).
- **IaC** (Infrastructure as Code): la **práctica** de definir esa infraestructura (sea IaaS, PaaS o lo que sea) en archivos de código versionados, en vez de clickear en el portal de Azure.

```hcl
# ejemplo Terraform (IaC) para un App Service
resource "azurerm_service_plan" "plan" {
  name                = "mi-app-service-plan"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  os_type             = "Linux"
  sku_name            = "P1v3"
}

resource "azurerm_linux_web_app" "app" {
  name                = "mi-api-java"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_service_plan.plan.location
  service_plan_id     = azurerm_service_plan.plan.id

  site_config {
    application_stack {
      java_version = "17"
    }
  }
}
```

Alternativas para Azure específicamente: **Bicep** (lenguaje nativo de Azure, más simple que ARM templates JSON) o **Terraform** (multi-cloud, el más usado en la industria en general).

```bicep
// ejemplo equivalente en Bicep
resource appServicePlan 'Microsoft.Web/serverfarms@2022-03-01' = {
  name: 'mi-app-service-plan'
  location: resourceGroup().location
  sku: { name: 'P1v3' }
}
```

## Por qué IaC importa para este puesto

Un cluster AKS, sus node pools, el ACR, las Network Policies — todo lo visto en [[KUBERNETES]] y en AKS se puede (y en un equipo serio, se **debe**) definir como código versionado, no crear a mano por CLI cada vez. Es lo que permite reproducir el mismo ambiente en dev/staging/prod de forma confiable, y es lo que un entrevistador espera que un Senior sepa nombrar y justificar aunque no sea el rol principal de Terraform/Bicep en el equipo.

## Errores comunes

- Confundir SLA con SLO — el SLA es el compromiso externo/contractual, el SLO es la meta interna (usualmente más estricta que el SLA).
- Pensar que "monitoring" y "observability" son sinónimos — monitoring es mirar dashboards conocidos; observability es poder responder preguntas nuevas sobre el sistema sin haber previsto esa pregunta de antemano (gracias a logs/metrics/traces ricos).
- Confundir IaaS con IaC (error de nomenclatura común, pero conceptualmente son cosas distintas: modelo de servicio vs práctica de gestión).
- Provisionar infraestructura de producción a mano por el portal de Azure "porque es más rápido", perdiendo reproducibilidad y trazabilidad de cambios.

## Checklist
- [ ] Puedo explicar SLI, SLO, SLA y Error Budget con un ejemplo propio.
- [ ] Conozco los tres pilares de observabilidad (logs, metrics, traces) y una herramienta de cada uno.
- [ ] Sé qué aporta Application Insights sobre lo que ya expone Spring Actuator.
- [ ] Distingo claramente IaaS de IaC.
- [ ] Puedo leer/explicar un archivo Terraform o Bicep básico de un recurso Azure.
