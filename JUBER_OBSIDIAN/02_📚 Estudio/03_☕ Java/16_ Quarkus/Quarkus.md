---
tags:
  - Java
  - Quarkus
  - CloudNative
Creado: 2026-08-09
Relacionado:
  - JAVA
  - Spring Boot
---

#Principal

# Quarkus

Framework Java de Red Hat, diseñado "Kubernetes-native" desde el inicio: arranque en milisegundos, bajo consumo de memoria, y compilación a binario nativo con GraalVM. Pensado para contenedores y serverless, donde el modelo tradicional de JVM (arranque lento, footprint grande) es un problema real.

## Módulos

| Orden | Tema | Estado | Nota |
|---|---|---|---|
| 1 | Fundamentos y Arquitectura | Pendiente | [[01 - Fundamentos y Arquitectura Quarkus]] |
| 2 | Quarkus vs Spring Boot | Pendiente | [[02 - Quarkus vs Spring Boot]] |
| 3 | CDI e Inyección de Dependencias | Pendiente | [[03 - CDI e Inyeccion de Dependencias]] |
| 4 | REST con RESTEasy Reactive | Pendiente | [[04 - REST con RESTEasy Reactive]] |
| 5 | Panache y Persistencia | Pendiente | [[05 - Panache y Persistencia]] |
| 6 | Configuración y Profiles | Pendiente | [[06 - Configuracion y Profiles]] |
| 7 | Testing en Quarkus | Pendiente | [[07 - Testing en Quarkus]] |
| 8 | Native Image con GraalVM | Pendiente | [[08 - Native Image con GraalVM]] |
| 9 | Swagger y OpenAPI | Pendiente | [[09 - Swagger y OpenAPI en Quarkus]] |

## Cuándo elegir Quarkus sobre Spring Boot

- Microservicios que necesitan arranque rápido (serverless, autoscaling agresivo en K8s).
- Footprint de memoria bajo es crítico (costos de infraestructura, muchas réplicas chicas).
- Ya usás Kubernetes como target primario de despliegue.

Spring Boot sigue siendo mejor opción cuando el ecosistema/librerías que necesitás son más maduras en Spring, o el equipo ya tiene expertise ahí — no es "Quarkus siempre gana", es una decisión de contexto (ver [[02 - Quarkus vs Spring Boot]]).

## Checklist general
- [ ] Entiendo por qué existe Quarkus y qué problema de Spring Boot ataca (arranque, footprint).
- [ ] Puedo crear un proyecto Quarkus con un endpoint REST funcionando.
- [ ] Entiendo CDI y cómo se diferencia del contenedor de Spring.
- [ ] Sé persistir datos con Panache.
- [ ] Sé compilar una imagen nativa con GraalVM y entiendo sus trade-offs.

## Referencias base
- [Quarkus Guides](https://quarkus.io/guides/)
- [Quarkus - Getting Started](https://quarkus.io/guides/getting-started)

---

## ✍️ Mis apuntes
> 📓 Apuntes del módulo → [[Apuntes - Quarkus]]

<!-- Anotaciones rápidas sobre el módulo en general -->

