---
tags:
  - Postulaciones
  - Arquitectura
  - SistemasDistribuidos
Creado: 2026-08-30
Relacionado:
  - RUTA FULL STACK ANGULAR NET AZURE
  - ARQUITECTURA
---

# Arquitectura y sistemas distribuidos

## Prioridad arquitectónica

Empezar con un **modular monolith** bien separado y desplegable. Pasar a microservicios cuando existan necesidades independientes de despliegue, escalado, ownership o aislamiento. La entrevista evalúa criterio, no la cantidad de patrones nombrados.

## Conceptos P0

- Separación de responsabilidades y dependencias.
- Capas, Clean Architecture pragmática y arquitectura hexagonal.
- Diseño de APIs, contratos y compatibilidad.
- DDD táctico ligero: entidades, value objects, agregados.
- Transacciones, concurrencia e idempotencia.
- Monolito modular vs microservicios.
- Escalado vertical/horizontal y estado compartido.

Repasar: [[Clean Architecture pragmatica]], [[Modular Monolith y boundaries]], [[Diseno de APIs empresariales]], [[DDD tactico entidades value objects agregados]].

## Conceptos P1

- Cache-Aside e invalidación.
- Mensajería asíncrona y delivery al menos una vez.
- Outbox e inbox/deduplicación.
- Retry con backoff, timeout y circuit breaker.
- Consistencia eventual.
- SAGA cuando una transacción cruza servicios.
- API Gateway y rate limiting.

Repasar: [[Caching MemoryCache Redis e invalidacion]], [[01 - Eventos Integracion y Outbox]], [[03 - Azure Service Bus]], [[03 - Resiliencia con Polly]].

## Método para responder system design

1. Aclarar usuarios, volumen, criticidad y restricciones.
2. Definir flujos y requisitos no funcionales.
3. Proponer la solución más simple que cumple.
4. Identificar datos, consistencia y ownership.
5. Diseñar API e integración.
6. Explicar seguridad.
7. Explicar fallos, reintentos e idempotencia.
8. Añadir observabilidad.
9. Describir despliegue y escalado.
10. Señalar trade-offs y evolución futura.

## Ejercicio

Diseñar una plataforma de capacitación multitenant:

- Angular para administración y alumnos.
- ASP.NET Core modular monolith.
- SQL Server/Azure SQL como sistema de registro.
- Blob Storage para archivos.
- Service Bus para emails y procesos largos.
- Redis solo para lecturas justificadas.
- App Service o Container Apps.
- OpenTelemetry y Application Insights.

Luego responder:

- ¿Qué ocurre si se publica dos veces el mismo mensaje?
- ¿Cómo se aísla un tenant?
- ¿Qué transacción no puede ser eventual?
- ¿Cuándo separarías un módulo en servicio?
- ¿Cómo migras el esquema sin romper la versión anterior?

## Criterio de salida

Puedes dibujar esta arquitectura en diez minutos y justificar cada componente, incluida la decisión de **no** usar MongoDB, AKS o microservicios donde no aportan valor.
