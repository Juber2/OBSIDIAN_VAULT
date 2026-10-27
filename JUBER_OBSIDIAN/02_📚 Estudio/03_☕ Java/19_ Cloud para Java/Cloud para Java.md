---
tags:
  - Java
  - Cloud
  - Azure
  - AWS
Creado: 2026-09-08
Relacionado:
  - Java
  - DevOps con Java
  - Azure
Categoria: Infraestructura
Estado:
  - Pendiente
Progreso: 0
Prioridad: Media-Alta
Nivel: Intermedio-Avanzado
Prerequisitos: Docker con Java, Spring Boot, CI CD para Java
---

#Principal

## 🧠 Por qué existe este módulo

Tu app Spring tiene que correr en algún lado, leer secretos de algún lado, publicar eventos en algún lado y ser observada desde algún lado. Ese "algún lado" es la nube, y es donde termina la mitad de las entrevistas de backend senior.

**Este módulo está escrito por concepto, no por proveedor.** Cada nota explica primero *el problema* (dónde guardo un secreto, cómo publico un evento, cómo sé que mi app está viva) y después muestra las dos implementaciones —**Azure** y **AWS**— lado a lado. Así aprendés una vez y respondés en cualquiera de las dos.

> [!info] Relación con el módulo Azure del vault
> Este módulo **no duplica** [[Azure]]: aquel es Azure en profundidad como plataforma. Este es *cómo se para una app Java encima de una nube*, con la comparación AWS al lado. Cuando un tema ya está cubierto allá, se enlaza en vez de repetirse.

## 🎯 Objetivos de Aprendizaje

- [ ] Elegir el servicio de cómputo correcto para una app Spring y justificarlo.
- [ ] Sacar todo secreto del código y del repositorio.
- [ ] Autenticar servicio-a-servicio sin credenciales embebidas.
- [ ] Publicar y consumir eventos con un broker gestionado.
- [ ] Instrumentar la app para que un incidente sea diagnosticable.
- [ ] Traducir cualquier arquitectura entre Azure y AWS.

## 🧠 Conceptos Fundamentales

- **La nube no es un servidor alquilado.** Lo que comprás es el servicio gestionado: parches, alta disponibilidad y escalado.
- **Identidad > credenciales.** Managed Identity (Azure) e IAM Roles (AWS) existen para que no haya passwords en ninguna parte.
- **Todo se cae.** Diseñás para que la instancia muera: sin estado en disco local, sin sesión en memoria.
- **El costo es una decisión de arquitectura**, no algo que se revisa a fin de mes.

## 📚 Temas

| #  | Tema | Estado | Nota |
|----|------|--------|------|
| 1  | Modelo mental — regiones, zonas, SLA y responsabilidad compartida | ⬜ | [[01 - Modelo Mental Cloud]] |
| 2  | Cómputo — dónde corre tu app Spring | ⬜ | [[02 - Computo para Apps Java]] |
| 3  | Contenedores y registries | ⬜ | [[03 - Contenedores y Registries]] |
| 4  | Configuración y secretos | ⬜ | [[04 - Configuracion y Secretos]] |
| 5  | Identidad y acceso sin credenciales | ⬜ | [[05 - Identidad y Acceso]] |
| 6  | Bases de datos gestionadas | ⬜ | [[06 - Bases de Datos Gestionadas]] |
| 7  | Mensajería gestionada | ⬜ | [[07 - Mensajeria Gestionada]] |
| 8  | Almacenamiento de objetos y archivos | ⬜ | [[08 - Almacenamiento de Objetos]] |
| 9  | Observabilidad en la nube | ⬜ | [[09 - Observabilidad Cloud]] |
| 10 | Red, exposición y API Management | ⬜ | [[10 - Red y Exposicion]] |
| 11 | Serverless con Java | ⬜ | [[11 - Serverless con Java]] |
| 12 | Infraestructura como código | ⬜ | [[12 - Infraestructura como Codigo]] |
| 13 | Costos y FinOps para backend | ⬜ | [[13 - Costos y FinOps]] |
| 14 | Tabla maestra de equivalencias Azure ↔ AWS | ⬜ | [[14 - Equivalencias Azure AWS]] |

## 🗺️ Mapa rápido de equivalencias

La tabla completa con criterios de elección está en [[14 - Equivalencias Azure AWS]]. Esta es la versión de bolsillo:

| Necesito… | Azure | AWS |
|---|---|---|
| Correr un JAR sin gestionar servidor | App Service | Elastic Beanstalk / App Runner |
| Correr contenedores gestionados | Container Apps | ECS Fargate |
| Kubernetes | AKS | EKS |
| Función efímera | Azure Functions | Lambda |
| Guardar secretos | Key Vault | Secrets Manager / Parameter Store |
| Identidad de la app | Managed Identity | IAM Role |
| Base relacional gestionada | Azure SQL / PostgreSQL Flexible | RDS / Aurora |
| NoSQL clave-valor | Cosmos DB | DynamoDB |
| Cola punto a punto | Service Bus Queue | SQS |
| Pub/sub | Service Bus Topic / Event Grid | SNS |
| Streaming de eventos | Event Hubs | Kinesis / MSK |
| Objetos y archivos | Blob Storage | S3 |
| Métricas, logs y trazas | Application Insights | CloudWatch + X-Ray |
| Puerta de enlace de APIs | APIM | API Gateway |
| Balanceador L7 | Application Gateway | ALB |
| Registro de imágenes | ACR | ECR |
| IaC nativo | Bicep / ARM | CloudFormation / CDK |

## 🔗 Relaciones
- **Prerequisitos**: [[01 - Docker con Java]], [[Spring Boot]], [[03 - CI CD para Java]]
- **Se conecta con**: [[Azure]], [[KUBERNETES]], [[DevOps con Java]], [[Sistemas Distribuidos]]
- **Se aplica en**: [[Fase 7 - Observabilidad y Despliegue]]

## 📌 Notas Rápidas
- Terraform funciona en las dos nubes: si vas a aprender un solo IaC, que sea ese.
- El patrón se transfiere, el nombre no. En entrevista importa que digas "necesito un almacén de secretos con rotación e identidad gestionada", no que recuerdes el nombre comercial.
- Para Java específicamente: arranque en frío y consumo de memoria de la JVM son *el* problema en serverless — ahí es donde entra [[Quarkus]] con GraalVM.

---

## ✍️ Mis apuntes
> 📓 Apuntes del módulo → [[Apuntes - Cloud para Java]]

<!-- Anotaciones rápidas sobre el módulo en general -->

