---
tags:
  - Azure
  - PaaS
Creado: 2026-08-12
Relacionado:
  - Azure
---

# App Service y Azure Functions

## Azure App Service: PaaS para apps web/API

Subís código (o una imagen de contenedor) y Azure gestiona el resto: SO, runtime, escalado, certificados TLS, dominios.

```bash
az webapp create \
  --resource-group mi-rg \
  --plan mi-app-service-plan \
  --name mi-api-java \
  --runtime "JAVA:17-java17"
```

Deploy de un JAR/WAR directo:
```bash
az webapp deploy --resource-group mi-rg --name mi-api-java --src-path target/mi-api.jar --type jar
```

O como contenedor (más flexible, mismo Dockerfile que usarías en cualquier lado):
```bash
az webapp config container set \
  --resource-group mi-rg \
  --name mi-api-java \
  --container-image-name miacr.azurecr.io/mi-api:1.0
```

## App Service Plan: el tamaño/tier que define costo y capacidad

```bash
az appservice plan create \
  --resource-group mi-rg \
  --name mi-app-service-plan \
  --sku P1V3 \
  --is-linux
```

El plan define cuántas instancias, qué tamaño de VM subyacente, y si soporta autoscaling — varias Web Apps pueden compartir el mismo plan.

## Deployment slots: deploy sin downtime

```bash
az webapp deployment slot create --resource-group mi-rg --name mi-api-java --slot staging
az webapp deploy --resource-group mi-rg --name mi-api-java --slot staging --src-path target/mi-api.jar --type jar
az webapp deployment slot swap --resource-group mi-rg --name mi-api-java --slot staging --target-slot production
```

Deployás a `staging`, probás, y hacés `swap` — el tráfico cambia de slot instantáneamente (blue-green deployment nativo de App Service, sin necesitar Kubernetes para lograrlo).

## Azure Functions: serverless por evento

```java
public class SaludoFunction {
    @FunctionName("saludo")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", methods = {HttpMethod.GET}, authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {
        return request.createResponseBuilder(HttpStatus.OK)
                .body("Hola desde Azure Functions")
                .build();
    }
}
```

## Triggers comunes

| Trigger | Se dispara cuando... |
|---|---|
| HTTP Trigger | llega un request HTTP |
| Timer Trigger | según un cron (ej. `0 0 3 * * *`) |
| Queue Trigger | llega un mensaje a Azure Storage Queue / Service Bus |
| Blob Trigger | se crea/modifica un archivo en Blob Storage |
| Cosmos DB Trigger | cambia un documento en Cosmos DB |

## Planes de hosting para Functions

| Plan | Modelo | Cuándo usarlo |
|---|---|---|
| Consumption | pagás solo por ejecución, escala a 0 | cargas esporádicas, cero tráfico = cero costo |
| Premium | siempre hay instancias "warm" | evita cold start, VNet integration |
| Dedicated (App Service Plan) | corre en un plan ya existente | control total, sin límites de tiempo de ejecución |

**Cold start** es la contra clásica del plan Consumption — la primera invocación después de inactividad tarda más porque la instancia se crea recién en ese momento. Con Java esto es más notorio que con lenguajes de arranque más liviano (mismo problema conceptual que JVM mode vs Native mode en Quarkus, ver [[08 - Native Image con GraalVM]]).

## Cuándo elegir App Service / Functions vs AKS

- **App Service**: una app monolítica o pocos servicios, sin necesidad de orquestación compleja entre ellos.
- **Functions**: lógica puntual disparada por eventos, cargas esporádicas donde pagar solo por ejecución tiene sentido económico.
- **AKS**: muchos microservicios que se comunican entre sí, necesitás control fino de networking/Ingress/Service Mesh, o portabilidad multi-cloud.

## Errores comunes

- Usar Functions con plan Consumption para algo con tráfico constante — termina costando más y con cold starts innecesarios; ahí conviene Premium o Dedicated.
- No usar deployment slots y hacer deploy directo a producción sin forma de rollback instantáneo.
- Elegir AKS "porque es lo que se usa en todos lados" para un caso que un simple App Service resolvería con mucho menos overhead operativo.

## Checklist
- [ ] Sé desplegar una app Java a App Service (JAR o contenedor).
- [ ] Entiendo deployment slots y su uso para blue-green sin Kubernetes.
- [ ] Sé escribir una Azure Function con al menos un trigger (HTTP, Timer o Queue).
- [ ] Conozco los planes de hosting de Functions y el trade-off de cold start.
- [ ] Tengo criterio para elegir entre App Service, Functions y AKS según el caso.
