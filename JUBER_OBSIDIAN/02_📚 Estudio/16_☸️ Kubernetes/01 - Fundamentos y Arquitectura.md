---
tags:
  - Kubernetes
Creado: 2026-08-09
Relacionado:
  - KUBERNETES
---

# Fundamentos y Arquitectura

Kubernetes (K8s) es un orquestador de contenedores: decide dónde corren, los reinicia si mueren, escala réplicas y gestiona red/storage entre ellos.

## Por qué no alcanza con Docker solo

Docker corre contenedores en **una** máquina. Cuando necesitás:
- Repartir contenedores entre varias máquinas.
- Reiniciar automáticamente uno que se cayó.
- Escalar réplicas según carga.
- Balancear tráfico entre ellas.

...necesitás un orquestador. Kubernetes es el estándar de facto.

## Arquitectura: Control Plane vs Nodos

```
┌─────────────── Control Plane ───────────────┐
│  API Server │ etcd │ Scheduler │ Controller  │
│                                Manager        │
└───────────────────────────────────────────────┘
              │ (comandos vía API)
┌────────────────────┬────────────────────┐
│   Nodo worker 1     │   Nodo worker 2    │
│  kubelet, kube-proxy│ kubelet, kube-proxy│
│  [Pod] [Pod]        │ [Pod] [Pod]        │
└────────────────────┴────────────────────┘
```

| Componente | Rol |
|---|---|
| **API Server** | puerta de entrada única; todo (kubectl, controllers) habla con él |
| **etcd** | base de datos clave-valor donde vive el estado del cluster |
| **Scheduler** | decide en qué nodo corre cada Pod nuevo |
| **Controller Manager** | reconcilia el estado real con el deseado (ej. si faltan réplicas, crea Pods) |
| **kubelet** | agente en cada nodo, arranca/vigila los Pods asignados |
| **kube-proxy** | maneja las reglas de red para que los Services funcionen |

## El modelo mental clave: estado deseado vs estado real

No le decís a Kubernetes "arrancá este contenedor". Le decís "quiero 3 réplicas de esta imagen corriendo siempre" (estado deseado), y el **Controller Manager** compara contra el estado real cada pocos segundos, corrigiendo diferencias (si un Pod muere, crea otro). Este loop de reconciliación es la base de todo lo demás en K8s.

## Unidad mínima: el Pod

Un **Pod** no es un contenedor — es un grupo de uno o más contenedores que comparten red y almacenamiento, y se despliegan juntos como una unidad atómica. La mayoría de las veces es 1 contenedor por Pod; múltiples contenedores en un Pod se usa para patrones como sidecar (ej. un proxy de logs junto a la app).

## Practicar local: Minikube / Kind

```bash
minikube start
kubectl get nodes
kubectl cluster-info
```

`kind` (Kubernetes IN Docker) es otra opción liviana, corre nodos como contenedores Docker — más rápido para CI.

## Comandos básicos de exploración

```bash
kubectl get pods
kubectl get nodes -o wide
kubectl describe pod <nombre>
kubectl logs <nombre-pod>
kubectl exec -it <nombre-pod> -- sh
```

## Errores comunes

- Confundir Pod con contenedor — un Pod puede tener varios contenedores.
- Pensar que Kubernetes "arranca contenedores" en vez de "reconcilia estado deseado" — esto cambia cómo debuggeás problemas.
- No diferenciar Control Plane (decide) de Nodos (ejecutan) al razonar dónde falló algo.

## Checklist
- [ ] Puedo explicar qué hace cada componente del Control Plane.
- [ ] Entiendo el loop de reconciliación (estado deseado vs real).
- [ ] Sé qué es un Pod y por qué no es lo mismo que un contenedor.
- [ ] Tengo un cluster local funcionando (Minikube o Kind) y ejecuté `kubectl get nodes`.
