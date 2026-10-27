---
tags:
  - Java
  - Cloud
  - Azure
  - AWS
  - Cheatsheet
Creado: 2026-09-08
Relacionado:
  - Cloud para Java
Palabras clave:
  - equivalencias, Azure, AWS, tabla maestra, traduccion, arquitectura
---

## 🧠 Para qué sirve esta nota

Tabla de traducción para leer una arquitectura escrita en una nube y entenderla en la otra. Es de **consulta**, no de estudio: el porqué de cada servicio está en las notas 01 a 13.

> [!tip] Lo que se transfiere es el patrón, no el nombre
> En una entrevista importa que digas *"necesito un almacén de secretos con rotación automática y acceso por identidad gestionada"*. Que se llame Key Vault o Secrets Manager es el detalle menos interesante de la frase.

## 🖥️ Cómputo

| Necesito | Azure | AWS |
|---|---|---|
| Máquina virtual | Virtual Machines | EC2 |
| Subir un JAR sin gestionar nada | App Service | Elastic Beanstalk |
| Contenedores sin servidores | **Container Apps** | **ECS Fargate** · App Runner |
| Contenedores sobre instancias propias | — | ECS en EC2 |
| Kubernetes gestionado | **AKS** | **EKS** |
| Funciones | Azure Functions | **Lambda** |
| Contenedor suelto y rápido | Container Instances | ECS Fargate (tarea única) |
| Escalado automático | Autoscale | Auto Scaling |
| Instancias con descuento e interrumpibles | Spot VMs | Spot Instances |

## 🗄️ Datos

| Necesito | Azure | AWS |
|---|---|---|
| PostgreSQL | Database for PostgreSQL Flexible Server | RDS PostgreSQL · **Aurora PostgreSQL** |
| MySQL | Database for MySQL Flexible Server | RDS MySQL · Aurora MySQL |
| SQL Server | **Azure SQL Database** | RDS SQL Server |
| NoSQL clave-valor | Cosmos DB | **DynamoDB** |
| Documental (Mongo) | Cosmos DB (API Mongo) | DocumentDB |
| Caché en memoria | Cache for Redis | ElastiCache · MemoryDB |
| Grafos | Cosmos DB (API Gremlin) | Neptune |
| Analítico | Synapse Analytics | Redshift |
| Búsqueda | AI Search | OpenSearch Service |
| Series temporales | Data Explorer | Timestream |
| Agrupador de conexiones | PgBouncer integrado | **RDS Proxy** |

## ✉️ Mensajería

| Necesito | Azure | AWS |
|---|---|---|
| Cola punto a punto | **Service Bus Queue** | **SQS** |
| Cola simple y barata | Storage Queue | SQS |
| Pub/sub | Service Bus Topic | **SNS** |
| Enrutado de eventos | Event Grid | **EventBridge** |
| Streaming de eventos | **Event Hubs** | Kinesis Data Streams |
| Kafka gestionado | Event Hubs (endpoint Kafka) | **MSK** |
| Orquestación de flujos | Logic Apps · Durable Functions | Step Functions |

## 📦 Almacenamiento

| Necesito | Azure | AWS |
|---|---|---|
| Objetos y archivos | **Blob Storage** | **S3** |
| Disco de máquina | Managed Disks | EBS |
| Sistema de archivos compartido | Files | EFS · FSx |
| Archivo de largo plazo | Blob Archive | Glacier Deep Archive |
| Inmutabilidad (WORM) | Immutable Storage | Object Lock |
| Acceso temporal a un objeto | SAS token | URL prefirmada |

## 🔐 Identidad y seguridad

| Necesito | Azure | AWS |
|---|---|---|
| Directorio de identidades | **Microsoft Entra ID** | IAM Identity Center |
| Identidad del recurso | **Managed Identity** | **IAM Role** |
| Identidad en Kubernetes | Workload Identity | IRSA |
| Permisos | Azure RBAC | políticas IAM |
| Almacén de secretos | **Key Vault** | **Secrets Manager** |
| Configuración barata | App Configuration | **Parameter Store** |
| Claves criptográficas | Key Vault · Managed HSM | **KMS** · CloudHSM |
| Certificados | Key Vault | ACM |
| Firewall de aplicación | WAF (en App Gateway/Front Door) | AWS WAF |
| Protección DDoS | DDoS Protection | Shield |
| Postura de seguridad | Defender for Cloud | Security Hub · GuardDuty |

## 🌐 Red

| Necesito | Azure | AWS |
|---|---|---|
| Red virtual | **VNet** | **VPC** |
| Firewall de instancia | NSG | Security Group |
| Firewall de subred | NSG | NACL |
| Balanceador L7 | **Application Gateway** | **ALB** |
| Balanceador L4 | Load Balancer | NLB |
| CDN y borde global | **Front Door** | **CloudFront** |
| Gestión de APIs | **APIM** | **API Gateway** |
| Acceso privado a servicios | Private Endpoint / Private Link | VPC Endpoint / PrivateLink |
| Salida a internet | NAT Gateway | NAT Gateway |
| DNS | Azure DNS | Route 53 |
| Firewall de red | Azure Firewall | Network Firewall |
| Conexión al datacenter propio | ExpressRoute · VPN Gateway | Direct Connect · Site-to-Site VPN |

## 📊 Observabilidad

| Necesito | Azure | AWS |
|---|---|---|
| Plataforma | **Azure Monitor** | **CloudWatch** |
| APM y trazas | **Application Insights** | X-Ray |
| Logs y consultas | Log Analytics (KQL) | CloudWatch Logs Insights |
| Métricas | Azure Monitor Metrics | CloudWatch Metrics |
| Paneles | Workbooks · Managed Grafana | CloudWatch Dashboards · Managed Grafana |
| Alertas | Azure Monitor Alerts | CloudWatch Alarms |
| OpenTelemetry | exportador a Azure Monitor | **ADOT** |
| Auditoría del plano de control | Activity Log | CloudTrail |

## 🚀 Entrega e infraestructura

| Necesito | Azure | AWS |
|---|---|---|
| Registro de imágenes | **ACR** | **ECR** |
| CI/CD | Azure DevOps · GitHub Actions | CodePipeline · GitHub Actions |
| IaC nativo | **Bicep** · ARM | **CloudFormation** · CDK |
| IaC multi-nube | Terraform | Terraform |
| Despliegue sin corte | Deployment Slots | despliegue azul/verde de ECS |
| Feature flags | App Configuration | AppConfig |

## 💰 Costos

| Necesito | Azure | AWS |
|---|---|---|
| Ver y analizar el gasto | Cost Management | Cost Explorer |
| Presupuestos y alertas | Budgets | Budgets |
| Compromiso con descuento | Reserved Instances · Savings Plans | Reserved Instances · Savings Plans |
| Recomendaciones | Advisor | Trusted Advisor · Compute Optimizer |
| Etiquetado | Tags | Tags |

## ⚖️ Diferencias que no son solo de nombre

> [!warning] Estas equivalencias no son simetrías perfectas
> Hay diferencias de modelo que importan al diseñar:

| Tema | Diferencia real |
|---|---|
| **Organización de recursos** | Azure agrupa en *resource groups* con ciclo de vida propio; AWS usa etiquetas y cuentas separadas |
| **Aurora** | no tiene equivalente directo en Azure: reimplementa el almacenamiento con 6 réplicas en 3 zonas |
| **Cosmos DB** | multi-modelo y multi-región con consistencia configurable en 5 niveles; DynamoDB es más simple y acotado |
| **Event Hubs** | habla el protocolo de Kafka de forma nativa; Kinesis tiene API propia y MSK **es** Kafka |
| **Security Group vs NSG** | ambos con estado, pero las NACL de AWS son sin estado y no tienen equivalente exacto |
| **Firewall de subred** | Azure usa el mismo NSG para instancia y subred; AWS separa Security Group y NACL |
| **Aislamiento de cuentas** | AWS separa entornos con cuentas distintas; Azure con suscripciones |

## 🔀 Traducir una arquitectura

Ejemplo del laboratorio `corebank`:

| Componente | Azure | AWS |
|---|---|---|
| API en contenedor | Container Apps | ECS Fargate |
| Imagen | ACR | ECR |
| Base de datos | PostgreSQL Flexible Server | RDS PostgreSQL |
| Eventos | Event Hubs (Kafka) | MSK |
| Cola de notificaciones | Service Bus Queue | SQS |
| Documentos | Blob Storage | S3 |
| Secretos | Key Vault | Secrets Manager |
| Identidad de la app | Managed Identity | IAM Task Role |
| Entrada | Application Gateway + WAF | ALB + WAF |
| Observabilidad | Application Insights | CloudWatch + X-Ray |
| IaC | Bicep o Terraform | CloudFormation o Terraform |

> [!tip] El código de la aplicación casi no cambia
> Con OpenTelemetry para observabilidad, Spring Cloud para configuración y secretos, y el protocolo de Kafka para eventos, lo que cambia entre nubes es **la configuración y el IaC**, no la lógica.
> Diseñar así no es "por si migramos": es lo que mantiene el código testeable en local y desacoplado del proveedor.

## 🧭 Qué es específico y qué es transferible

| Transferible ✅ | Específico ⚠️ |
|---|---|
| Modelo de identidad sin credenciales | sintaxis de las políticas |
| Endpoints privados | nombres y configuración |
| Colas, pub/sub y streaming | límites y semántica de cada servicio |
| Terraform | proveedores y recursos |
| OpenTelemetry | exportador y destino |
| Contenedores | definición de servicio |
| Zonas y regiones | disponibilidad por región |
| Ciclo de vida del almacenamiento | nombres de los niveles |

## 📌 Notas Rápidas
- Aprendé **el patrón**: almacén de secretos, identidad del recurso, endpoint privado, cola vs stream, ciclo de vida del almacenamiento. Los nombres comerciales se buscan en un minuto; el criterio no.
- La diferencia entre Security Group (con estado) y NACL (sin estado) es la trampa más frecuente al pasar de Azure a AWS: se permite la entrada y la respuesta no sale.
- Terraform y OpenTelemetry son las dos herramientas que más reducen el acoplamiento al proveedor. Si tuviera que priorizar dos cosas de este módulo para aprender bien, serían esas.
- Los servicios cambian de nombre con cierta frecuencia (Azure AD → Entra ID, por ejemplo). Verificá los nombres actuales antes de una entrevista.

## 🔗 Relaciones
- **Módulo**: [[Cloud para Java]] — última nota del módulo
- **Anterior**: [[13 - Costos y FinOps]]
- **Se conecta con**: [[Azure]], [[KUBERNETES]], [[DevOps con Java]]

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

