---
tags:
  - Azure
  - Laboratorio
  - Portafolio
Creado: 2026-08-30
Relacionado:
  - Azure
  - Laboratorios
---

# Laboratorio Integrado — API de Órdenes

Un solo proyecto que cubre casi todo el anuncio de **Entelgy** (prioridad máxima) y buena parte de las demás vacantes del radar. La idea no es aprender 10 servicios sueltos: es tener **una cosa desplegada que puedas mostrar y defender** — que es la única evidencia que [[01 - Diagnóstico y matriz de competencias]] considera válida.

> Sirve igual en .NET o en Java/Spring. La arquitectura y los servicios de Azure son idénticos; solo cambia el SDK. Si vas a las vacantes políglotas, construirlo en un stack y saber explicar cómo sería en el otro vale más que dos proyectos a medias.

## Arquitectura objetivo

```
                    [ Entra ID ]
                         │ valida tokens
                         ▼
Cliente ──► [ App Service: API de Órdenes ]
                    │           │
        escribe     │           │  publica evento
                    ▼           ▼
            [ Azure SQL ]  [ Service Bus: ordenes ]
                    ▲                   │
        lee (SAS)   │                   ▼
            [ Blob Storage ]   [ Azure Function: facturación ]
                                        │ consumidor idempotente
                                        ▼
                                  [ Azure SQL ]

Transversal:  [ Key Vault ] ── secretos      [ App Insights ] ── trazas correlacionadas
Entrega:      [ GitHub Actions ] ── build → test → deploy
```

## Fase 1 — La API funcionando en local

**Objetivo:** un CRUD de órdenes que arranca sin nada de Azure.

- Separar dominio, casos de uso e infraestructura (Clean Architecture pragmática — sin crear 7 proyectos para un CRUD).
- Persistencia con EF Core (o JPA/Hibernate en Java).
- Validación de entrada y errores consistentes: `ProblemDetails` en .NET, `@ControllerAdvice` en Java.
- OpenAPI/Swagger funcionando.
- Tests: unitarios del dominio + un test de integración con base real (Testcontainers).

✅ **Checkpoint:** `docker compose up` levanta API + base y los tests pasan en limpio.

> No sigas a la fase 2 sin tests. Agregarlos después de tener 6 servicios de Azure encima es mucho más caro, y "tengo tests de integración reales" es un diferenciador concreto en entrevista.

## Fase 2 — Contenedor y base en Azure

- Dockerfile **multi-stage** (build en una imagen, runtime en otra mínima).
- Subir la imagen a Azure Container Registry.
- Crear Azure SQL y ejecutar las migraciones.
- Desplegar en App Service.

```bash
az acr build --registry miacr --image ordenes-api:v1 .

az webapp create --resource-group mi-rg --plan mi-plan --name ordenes-api \
  --deployment-container-image-name miacr.azurecr.io/ordenes-api:v1
```

✅ **Checkpoint:** la API responde desde una URL pública y Swagger carga.

Ver [[03 - App Service y Azure Functions]] y [[05 - Azure SQL y Blob Storage]].

## Fase 3 — Secretos y conexión sin passwords

- Managed Identity en el App Service.
- Key Vault con RBAC y rol `Key Vault Secrets User`.
- **Meta real:** conectarse a Azure SQL con Managed Identity, sin password en ningún lado.

✅ **Checkpoint:** no queda ni una credencial en el repo, en el pipeline ni en la configuración del App Service. Podés explicar la cadena completa App Service → Entra ID → Key Vault / SQL.

Ver [[01 - Key Vault y Managed Identity]].

## Fase 4 — Seguridad con Entra ID

- Dos app registrations: la API y el cliente.
- Exponer un scope (`Ordenes.Read`, `Ordenes.Write`).
- Validar firma, issuer, audience y expiración.
- Policies por scope **y** autorización por recurso: un usuario no puede leer la orden de otro.

✅ **Checkpoint:** un token sin el scope correcto devuelve 403, y un token válido de *otro* usuario tampoco puede leer una orden ajena. **Probá los dos casos** — el segundo es el que casi nadie implementa y el que corresponde a A01 de OWASP.

Ver [[04 - Entra ID y Seguridad de APIs]].

## Fase 5 — Mensajería y consumidor idempotente

- Al confirmar una orden, publicar un evento a Service Bus.
- Azure Function que lo consume y genera la factura.
- El consumidor debe ser **idempotente**: registrar el `MessageId` procesado en la misma transacción que el efecto.
- Configurar `MaxDeliveryCount` y revisar la DLQ.

✅ **Checkpoint:** publicá el **mismo mensaje dos veces a propósito** y verificá que se genera una sola factura. Ese experimento es la respuesta viva a la pregunta de Entelgy sobre duplicados.

**Extra de alto valor:** implementar **Outbox** — guardar el evento en una tabla dentro de la transacción de la orden, y publicarlo desde un proceso aparte. Es lo que convierte el proyecto de "usa una cola" a "entiende consistencia distribuida".

Ver [[02 - Service Bus y Mensajeria]].

## Fase 6 — Observabilidad

- Application Insights en API y Function.
- Logs estructurados con `OrdenId` y `ClienteId` como campos.
- Verificar que el `operation_Id` se propaga **API → Service Bus → Function**.
- Un dashboard con tasa de error, latencia p95 y profundidad de cola.
- Una alerta accionable.

✅ **Checkpoint:** una sola consulta KQL por `operation_Id` reconstruye el recorrido completo de una orden atravesando los tres servicios.

Ver [[03 - Application Insights y Azure Monitor]].

## Fase 7 — CI/CD

```
pull request → build → tests → análisis estático → imagen → deploy a staging
             → smoke test → swap de slot a producción
```

- GitHub Actions con **OIDC/federated credentials** hacia Azure — sin secretos de larga duración.
- Deployment slots: deploy a `staging`, smoke test, `swap`.
- Rollback = swap inverso.

✅ **Checkpoint:** un push a main despliega solo, y podés hacer rollback en un comando.

Ver [[GitHub Actions CI CD]].

## Fase 8 — Lo que lo vuelve defendible en entrevista

Esto es lo que separa un repo más de una evidencia profesional:

- **README ejecutable**: alguien clona y lo levanta siguiendo los pasos, sin preguntarte nada.
- **Diagrama** de la arquitectura (el de arriba sirve).
- **2-3 ADR** cortos explicando decisiones y alternativas descartadas:
  - por qué Clean Architecture pragmática y no capas tradicionales;
  - por qué Outbox en vez de publicar directo tras el commit;
  - por qué Managed Identity en vez de secretos en Key Vault.
- **Demo de 5 minutos** ensayada: crear orden → ver evento → ver factura → mostrar la traza correlacionada.

> Un ADR que explica **qué descartaste y por qué** vale más que tres features extra. Es la evidencia de criterio, que es justo lo que evalúan en un senior.

## Qué requisitos del radar cubre este único proyecto

| Requisito | Apariciones en el radar | Cubierto en |
|---|---:|---|
| REST/JSON e integraciones | 10/14 | Fase 1 |
| Bases relacionales | 11/14 | Fases 1-2 |
| Seguridad OAuth/JWT | 7/14 | Fase 4 |
| CI/CD | 7/14 | Fase 7 |
| Microservicios | 7/14 | Fases 5-6 |
| Testing | 6/14 | Fase 1 |
| Observabilidad | 4/14 | Fase 6 |
| Mensajería / DLQ | 4/14 | Fase 5 |
| Azure | 3/14 | Fases 2-7 |
| Docker | transversal | Fase 2 |
| Clean Architecture / DDD / CQRS | 3/14 | Fases 1 y 8 |

Un proyecto, once requisitos que se repiten en el mercado. Ese es el argumento para construirlo antes que estudiar otra tecnología suelta.

## Errores comunes al hacer este laboratorio

- Empezar por Azure y dejar los tests para el final. Se vuelven caros y terminan sin hacerse.
- Crear 7 proyectos de Clean Architecture para un CRUD — sobreingeniería que en entrevista juega en contra.
- Desplegar sin observabilidad: cuando algo falla en la nube, sin trazas estás ciego.
- Dejar credenciales "temporales" en el pipeline que nunca se quitan.
- No probar nunca el mensaje duplicado, y afirmar que el consumidor es idempotente.
- Tener el proyecto pero no poder explicar **por qué** cada decisión, que es lo único que preguntan.

## Checklist final

- [ ] Corre en local con un comando y los tests pasan.
- [ ] Está desplegado y accesible públicamente.
- [ ] No hay ninguna credencial almacenada en ningún lado.
- [ ] Un token sin scope da 403; un usuario no accede a recursos de otro.
- [ ] Un mensaje duplicado no genera efectos duplicados, y lo verifiqué.
- [ ] Una consulta reconstruye una operación completa entre los tres servicios.
- [ ] Un push despliega y sé hacer rollback.
- [ ] Tengo README, diagrama, ADR y una demo de 5 minutos ensayada.
