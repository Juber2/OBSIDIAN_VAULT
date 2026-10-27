---
tags:
  - Azure
  - Laboratorios
Creado: 2026-08-30
Relacionado:
  - Azure
---

#Principal

# Laboratorios Azure

Los [[Conceptos]] explican **qué es** cada servicio. Estos laboratorios cubren **cómo se usa**: comandos, código en .NET y Java, y las preguntas de entrevista que cada tema dispara.

Creados a partir del hueco detectado el 2026-08-30: Key Vault, Service Bus, Application Insights, Azure SQL y Entra ID solo aparecían como menciones de paso dentro de notas de otro tema, y Entra ID no aparecía en absoluto — pese a que [[Entelgy - Backend NET Azure]] (prioridad máxima) los exige explícitamente.

| Orden | Tema | Estado | Nota |
|---|---|---|---|
| 1 | Key Vault y Managed Identity | Pendiente | [[01 - Key Vault y Managed Identity]] |
| 2 | Service Bus y Mensajería | Pendiente | [[02 - Service Bus y Mensajeria]] |
| 3 | Application Insights y Azure Monitor | Pendiente | [[03 - Application Insights y Azure Monitor]] |
| 4 | Entra ID y Seguridad de APIs | Pendiente | [[04 - Entra ID y Seguridad de APIs]] |
| 5 | Azure SQL y Blob Storage | Pendiente | [[05 - Azure SQL y Blob Storage]] |
| 6 | Laboratorio Integrado — API de Órdenes | Pendiente | [[06 - Laboratorio Integrado - API de Ordenes]] |

## Cómo usarlos

Las notas 1-5 son de referencia: se leen sueltas según lo que pida cada vacante. La nota 6 las ata en **un solo proyecto desplegable** — es la que produce evidencia mostrable, y la que conviene hacer aunque no leas las demás en orden.

## Preguntas de entrevista que cubren

Estas salen textuales de [[Entelgy - Backend NET Azure]]:

| Pregunta | Nota |
|---|---|
| ¿Cómo accede App Service a Key Vault sin secretos? | [[01 - Key Vault y Managed Identity]] |
| ¿Cómo procesas mensajes de Service Bus sin duplicar efectos? | [[02 - Service Bus y Mensajeria]] |
| ¿Qué observas en Application Insights durante un incidente? | [[03 - Application Insights y Azure Monitor]] |
| ¿Cuál es la diferencia entre OAuth2, OIDC y JWT? | [[04 - Entra ID y Seguridad de APIs]] |
| ¿Cómo proteges una API con Entra ID? | [[04 - Entra ID y Seguridad de APIs]] |
| ¿Cómo harías rollback? | [[06 - Laboratorio Integrado - API de Ordenes]] (fase 7) |
