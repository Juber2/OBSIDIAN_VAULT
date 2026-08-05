---
tags:
  - Docker
  - DevOps
  - Backend
  - Libro
Creado: 2026-07-28
Relacionado:
  - ESTUDIO
---

#Principal

# Libro Docker Junior a Senior

Esta carpeta no es solo una ruta. Esta pensada como un **libro de estudio practico** para avanzar desde bases de contenedores hasta criterio senior de infraestructura con Docker.

La estructura esta dividida en capitulos. Cada capitulo contiene explicacion, modelo mental, ejemplos, errores comunes, practica, preguntas de repaso y criterios junior/mid/senior.

## Capitulos

| Orden | Capitulo | Estado | Nota |
|---|---|---|---|
| 1 | Fundamentos de Contenedores | Pendiente | [[Fundamentos de Contenedores]] |
| 2 | Imagenes y Dockerfile | Pendiente | [[Imagenes y Dockerfile]] |
| 3 | Volumenes y Persistencia | Pendiente | [[Volumenes y Persistencia]] |
| 4 | Redes | Pendiente | [[Redes]] |
| 5 | Docker Compose | Pendiente | [[Docker Compose]] |
| 6 | Seguridad | Pendiente | [[Seguridad]] |
| 7 | Optimizacion de Imagenes | Pendiente | [[Optimizacion de Imagenes]] |
| 8 | Registries y Distribucion | Pendiente | [[Registries y Distribucion]] |
| 9 | CI CD | Pendiente | [[CI CD]] |
| 10 | Orquestacion Basica | Pendiente | [[Orquestacion Basica]] |
| 11 | Monitoreo y Logging | Pendiente | [[Monitoreo y Logging]] |
| 12 | Docker para Diferentes Stacks | Pendiente | [[Docker para Diferentes Stacks]] |
| 13 | Senior Docker | Pendiente | [[Senior Docker]] |
| 14 | Proyectos Docker | Pendiente | [[Proyectos]] |

## Como usar este libro

1. Lee el capitulo en orden.
2. Ejecuta al menos un ejemplo en un proyecto Docker real de practica.
3. Convierte cada tema importante en un mini commit o en un cambio real a un Dockerfile/Compose de un proyecto tuyo.
4. Aplica lo aprendido a un proyecto real: [[NESTJS|el backend NestJS]] o StaffTrainingPlatform (.NET) son buenos candidatos.
5. Vuelve a cada nota y marca el checklist solo cuando puedas explicarlo sin mirar tutorial.

## Resultado esperado

Al terminar, deberias poder construir, asegurar, desplegar y operar aplicaciones containerizadas reales: imagenes optimizadas y multi-stage, redes y volumenes correctamente diseñados, stacks completos con Compose, pipelines de CI/CD que build/testean/publican imagenes, orquestacion basica, observabilidad de containers en produccion, y criterio senior de arquitectura, costos y seguridad.

## Checkpoint senior

- [ ] Puedo escribir Dockerfiles multi-stage optimizados, seguros (usuario no-root, capabilities minimas) y con buen aprovechamiento de cache.
- [ ] Domino volumenes y redes sin caer en perdida de datos ni exposicion innecesaria de puertos.
- [ ] Puedo diseñar un stack completo con Docker Compose, con segmentacion de red y health checks reales.
- [ ] Puedo escanear, firmar y distribuir imagenes de forma segura en un registry, con tagging trazable.
- [ ] Puedo integrar Docker en un pipeline de CI/CD completo, con cache y testing dentro de containers.
- [ ] Puedo decidir con criterio entre Compose, Swarm y Kubernetes segun el contexto real del proyecto, no por moda.
- [ ] Puedo diagnosticar problemas de produccion (logs, metricas, cgroups, namespaces) y disenar despliegues de bajo riesgo.

## Referencias base

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Specification](https://docs.docker.com/compose/compose-file/)
- [OCI (Open Container Initiative)](https://opencontainers.org/)
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
