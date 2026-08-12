---
tags:
  - Azure
  - CloudNative
Creado: 2026-08-12
Relacionado:
  - Azure
---

# PaaS vs IaaS vs Cloud Native

## Los tres modelos, en orden de "cuánto gestiona Azure por vos"

| Modelo | Qué gestionás vos | Qué gestiona Azure | Ejemplo |
|---|---|---|---|
| **IaaS** (Infrastructure as a Service) | SO, runtime, escalado, parches | solo el hardware/virtualización | Azure VMs |
| **PaaS** (Platform as a Service) | tu código y config de la app | SO, runtime, escalado, patching | App Service, Azure Functions, Azure SQL Database |
| **Cloud Native** | diseño de la app para aprovechar el modelo distribuido | infraestructura + orquestación | AKS, Container Apps, Service Bus, Cosmos DB |

**IaaS** = alquilar una máquina y administrarla vos (más control, más trabajo operativo).
**PaaS** = subir código, Azure administra todo lo demás (menos control, menos trabajo operativo).
**Cloud Native** no es un servicio específico — es un **enfoque de diseño**: apps construidas para correr distribuidas, escalar horizontalmente, tolerar fallos de instancias individuales, y desplegarse en contenedores/orquestadores. Se apoya en PaaS y en servicios administrados (colas, bases de datos gestionadas), evitando estado en el propio proceso.

## Los 12-factor app principles (base conceptual de Cloud Native)

Un resumen de los que más se preguntan en entrevista:
1. **Config en el entorno**, no en el código (variables de entorno, no hardcodeado).
2. **Stateless**: la app no guarda estado local que sobreviva a un restart — el estado va a una base de datos o cache externa.
3. **Disposability**: arranque rápido, apagado gracioso — clave para autoscaling y rolling updates.
4. **Logs como stream de eventos**, no archivos locales — se centralizan (Azure Monitor, ELK, etc.).

Esto conecta directo con por qué Kubernetes/AKS existe: si tu app cumple estos principios, un orquestador puede matarla y recrearla en cualquier nodo sin pérdida de datos.

## Servicios PaaS de Azure más relevantes para un backend Java

| Servicio | Para qué |
|---|---|
| **App Service** | hosting PaaS para apps web/API, sin gestionar VMs ni contenedores manualmente |
| **Azure Functions** | serverless, ejecuta código por evento (HTTP, cola, timer), pagás solo por ejecución |
| **Azure Container Apps** | corre contenedores sin gestionar un cluster completo de Kubernetes (más simple que AKS) |
| **AKS (Azure Kubernetes Service)** | Kubernetes completo, gestionado — control plane administrado por Azure, vos gestionás los nodos worker |
| **Azure SQL Database / Cosmos DB** | bases de datos PaaS, sin administrar el motor manualmente |
| **Azure Service Bus** | mensajería/colas gestionada (alternativa PaaS a RabbitMQ/Kafka autogestionado) |

## AKS vs Container Apps vs App Service para contenedores

Pregunta común de entrevista: "¿por qué usarías AKS y no algo más simple?"

- **App Service (contenedor único)**: la app más simple, un solo contenedor, sin necesidad de orquestación real.
- **Container Apps**: varios microservicios en contenedores, con autoscaling y Dapr integrado, pero sin necesitar control total sobre el cluster (Azure abstrae Kubernetes por debajo).
- **AKS**: cuando necesitás control total del cluster (Ingress custom, Service Mesh, políticas de red específicas, multi-tenant, portabilidad entre clouds) — es el que pide el puesto que estás preparando, típico en arquitecturas de microservicios complejas y equipos con expertise en Kubernetes.

## Errores comunes (conceptuales, para entrevista)

- Decir "Cloud Native = usar la nube" — es un malentendido común; Cloud Native es sobre el **diseño** de la app (stateless, distribuida, resiliente), no solo dónde corre.
- Confundir PaaS con Cloud Native como sinónimos — PaaS es un modelo de servicio, Cloud Native es un enfoque arquitectónico que puede apoyarse en PaaS.
- No poder justificar cuándo AKS es la elección correcta vs una alternativa más simple (App Service, Container Apps) — mostrar criterio, no "porque es lo que sé".

## Checklist
- [ ] Puedo explicar la diferencia entre IaaS, PaaS y Cloud Native con ejemplos concretos de Azure.
- [ ] Conozco los 12-factor principles y por qué importan para apps en Kubernetes.
- [ ] Sé argumentar cuándo elegir AKS vs Container Apps vs App Service.
- [ ] Puedo nombrar al menos 5 servicios PaaS de Azure relevantes para un backend Java.
