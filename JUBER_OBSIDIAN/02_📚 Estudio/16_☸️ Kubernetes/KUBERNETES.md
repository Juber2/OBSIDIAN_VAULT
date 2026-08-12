---
tags:
  - Kubernetes
  - DevOps
  - Backend
Creado: 2026-08-09
Relacionado:
  - ESTUDIO
  - DOCKER
---

#Principal

# Kubernetes — Ruta de Estudio

Continuación natural de [[DOCKER|Docker]]: acá se orquesta lo que ahí se contenedoriza.

**Estado de esta ruta: módulos 1-6 completos (contenido listo). Módulos 7-12 aún no creados.**

## Roadmap

| Orden | Módulo | Estado | Nota |
|---|---|---|---|
| 1 | Fundamentos y Arquitectura | Pendiente (contenido listo) | [[01 - Fundamentos y Arquitectura]] |
| 2 | Pods, Deployments y ReplicaSets | Pendiente (contenido listo) | [[02 - Pods Deployments y ReplicaSets]] |
| 3 | Services y Networking | Pendiente (contenido listo) | [[03 - Services y Networking]] |
| 4 | ConfigMaps, Secrets y Volúmenes | Pendiente (contenido listo) | [[04 - ConfigMaps Secrets y Volumenes]] |
| 5 | Ingress y Exposición Externa | Pendiente (contenido listo) | [[05 - Ingress y Exposicion Externa]] |
| 6 | Namespaces, RBAC y Seguridad Básica | Pendiente (contenido listo) | [[06 - Namespaces RBAC y Seguridad Basica]] |
| 7 | Helm y Gestión de Paquetes | No creado | — |
| 8 | StatefulSets y Storage Avanzado | No creado | — |
| 9 | Autoscaling (HPA/VPA) y Resource Management | No creado | — |
| 10 | Observabilidad (Metrics, Logging, Probes) | No creado | — |
| 11 | CI/CD hacia Kubernetes (GitOps) | No creado | — |
| 12 | Senior Kubernetes (multi-cluster, costos, troubleshooting) | No creado | — |

## Cómo usar esta ruta

1. Leé los módulos en orden — cada uno asume conceptos del anterior.
2. Practicá con Minikube, Kind o Docker Desktop (Kubernetes local) antes de tocar un cluster real.
3. Cuando se completen los módulos 7-12, se agregan siguiendo el mismo formato.

## Checklist general (parcial, hasta módulo 6)
- [ ] Entiendo la arquitectura de un cluster (control plane vs nodos).
- [ ] Sé crear y gestionar Pods/Deployments/ReplicaSets.
- [ ] Entiendo Services y cómo se resuelve el networking interno.
- [ ] Sé manejar configuración y secretos sin hardcodearlos en la imagen.
- [ ] Sé exponer una app al exterior con Ingress.
- [ ] Entiendo namespaces y aislamiento básico de permisos (RBAC).

## Relacionado con el puesto que estás preparando
[[Postulacion|Senior Java Developer - Coforge/BCP]] pide explícitamente Kubernetes + AKS. Este roadmap es la base; el módulo de [[Azure|Azure]] cubre la parte específica de AKS.

## Referencias base
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
