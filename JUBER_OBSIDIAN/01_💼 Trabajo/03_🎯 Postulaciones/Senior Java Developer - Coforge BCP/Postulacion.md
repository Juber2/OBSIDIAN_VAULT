---
tags:
  - Postulacion
  - Java
Creado: 2026-08-12
Relacionado:
  - JAVA
  - Azure
---

#Principal

# Senior Java Developer — Coforge (vía Encora) / Proyecto BCP

- **Empresa que contrata:** Coforge, para proyecto con BCP (vía Encora Inc.)
- **Puesto:** Senior Java Developer (#22515)
- **Ubicación:** Lima, Perú — Híbrido, 2 veces/semana en Chorrillos y/o La Molina
- **Publicado:** 11-08-2026
- **Estado postulación:** 🔴 no postulado aún — preparando conocimientos primero

## Habilidades clave del puesto
Java, Spring, Quarkus, Microservicios, REST/RESTful, Docker, Kubernetes, Azure, Cloud Native

## Requisitos indispensables — gap analysis (2026-08-12)

| Requisito | Estado en el vault | Nota |
|---|---|---|
| Java JDK 8/11/17 | 🟡 parcial | [[JAVA]] — carpetas creadas, verificar profundidad real de contenido |
| Spring | 🟢 cubierto | [[Spring Boot]], [[Spring Core]], [[Spring Data]], [[Spring Security]] |
| Quarkus | 🟢 cubierto | [[Quarkus]] |
| REST/RESTful | 🟢 cubierto | dentro de Spring Boot y Quarkus |
| Swagger | 🟢 cubierto | [[09 - Swagger y OpenAPI]] (Spring), [[09 - Swagger y OpenAPI en Quarkus]] |
| Maven / Gradle | 🟢 cubierto | [[Build Tools]] |
| Docker | 🟢 cubierto | [[DOCKER]] — 14 capítulos |
| Kubernetes | 🟢 cubierto | [[KUBERNETES]] — módulos 1-6 completos |
| AKS (Azure Kubernetes Service) | 🟢 cubierto | [[02 - AKS Fundamentos]] |
| Cloud Native / PaaS Azure | 🟢 cubierto | [[01 - PaaS vs IaaS vs Cloud Native]], [[03 - App Service y Azure Functions]] |
| Arquitectura: capas, SAGA, Cache Aside | 🟢 cubierto | [[ARQUITECTURA]] — sección SAGA y Cache Aside agregada |
| NoSQL: tablas, índices, ACID vs BASE | 🟢 cubierto (reescrita 2026-08-12) | [[05 - NoSQL desde Java]] — la versión original de esta nota NO cubría ACID vs BASE ni índices, la reemplacé por una completa |

**Los 12 requisitos indispensables están cubiertos con contenido.**

## Verificación honesta (2026-08-12)
Antes de dar por cerrado esto, releí el contenido real (no solo el índice/checklist) de cada nota marcada 🟢:
- ❌ **NoSQL** estaba mal marcada — la nota original era genérica (47 líneas) y no mencionaba ACID/BASE ni índices, que el puesto pide textual. **Corregido.**
- ✅ **Java base** (`01_Fundamentos`, `02_Intermedio`, `03_Avanzado`): completado 2026-08-12. Las 28 notas tenían "📌 Notas Rápidas" vacía y 4 también "⚠️ Errores Comunes" vacía — se llenaron todas con contenido real (tips, gotchas, warnings con código).
- Todo lo demás (Kubernetes, Azure/AKS, SAGA/Cache Aside, Swagger, Docker, Spring, Quarkus) lo escribí yo en esta sesión con profundidad completa, así que no tenía el mismo riesgo de estar "a medias".

## Requisitos deseables

| Requisito | Estado |
|---|---|
| APIM, Azure Application Gateway | 🟢 cubierto | [[04 - APIM y Application Gateway]] |
| App Service, Functions | 🟢 cubierto | [[03 - App Service y Azure Functions]] |
| Data Factory | 🟢 cubierto | [[05 - Data Factory y Conceptos Data Warehouse]] |
| Data Warehouse: OLAP, OLTP, DataMart, Data Vault | 🟢 cubierto | [[05 - Data Factory y Conceptos Data Warehouse]] |
| SRE, APM, IaaS, IaC | 🟢 cubierto | [[06 - SRE APM y IaC]] |

## Plan de preparación (completado 2026-08-12)
1. [x] Terminar [[KUBERNETES]] módulos 3-6
2. [x] Construir módulo Azure Cloud Native + AKS
3. [x] Nota de patrones SAGA y Cache Aside
4. [x] Swagger/OpenAPI en Java (Spring y Quarkus)
5. [x] Deseables: Azure PaaS avanzado, Data Warehouse, SRE/APM/IaC

## Siguiente paso real
Todo el contenido está armado. Ahora toca **estudiarlo y practicar** — no queda más para "crear", queda para "aprender". Sugerencia: repasar 1-2 notas por día, y para cada requisito indispensable poder explicarlo en voz alta sin mirar la nota antes de postular.

## Preguntas reales de entrevista (referencia de un postulante, 2026-08-12)
El usuario compartió temas que le preguntaron a otro candidato para este mismo tipo de puesto. Cruzados contra el vault:

| Tema preguntado | Estado |
|---|---|
| SAGA | 🟢 [[ARQUITECTURA]] |
| Arquitectura Hexagonal | 🟢 ya existía [[03 - Arquitectura Hexagonal]], + resumen agregado en [[ARQUITECTURA]] |
| Arquitectura en Capas | 🟢 agregada en [[ARQUITECTURA]] (no existía explícita) |
| Circuit Breaker | 🟢 ya existía, completé Notas Rápidas |
| Factory, Singleton, Strategy | 🟢 ya existían en [[Patrones de Diseño]] (15 patrones GoF completos), completé Notas Rápidas en los 15 |
| Repository (pattern) | 🟢 ya cubierto vía [[03 - Repositorios]] (Spring Data) |
| Lombok | 🟢 nota nueva [[06 - Lombok]] (no existía) |
| Quarkus | 🟢 [[Quarkus]] |
| JPA | 🟢 [[01 - JPA e Hibernate]] |
| Spring Boot | 🟢 [[Spring Boot]] |
| Kafka | 🟢 [[01 - Kafka Fundamentos]], [[02 - Kafka con Spring Boot]] |
| Redis | 🟢 [[05 - Cache con Redis y Caffeine]] |

**Actualización 2026-08-12 (tarde):** completado también el resto del vault de Java a pedido del usuario. Las 46 notas restantes de Build Tools, Testing, Spring Boot/Core/Data/Security, Mensajería (Kafka/RabbitMQ) y Microservicios ya tienen "Notas Rápidas" con contenido real. Verificado con script: **0 notas vacías en toda la carpeta `03_☕ Java`.**

## Checklist antes de postular
- [ ] Cubrir los 12 requisitos indispensables al menos a nivel "puedo hablar de esto con confianza en entrevista".
- [ ] Repasar CV y alinearlo con las palabras clave del puesto (Cloud Native, AKS, SAGA, Cache Aside).
- [ ] Preparar 2-3 ejemplos concretos de proyectos propios que toquen Docker/Kubernetes/microservicios.
- [ ] Postular vía el link original (botón "Solicitar").
