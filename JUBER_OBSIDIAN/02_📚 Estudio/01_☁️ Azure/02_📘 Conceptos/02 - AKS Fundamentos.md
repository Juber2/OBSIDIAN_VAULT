---
tags:
  - Azure
  - AKS
  - Kubernetes
Creado: 2026-08-12
Relacionado:
  - Azure
  - KUBERNETES
---

# AKS — Azure Kubernetes Service

Prerequisito: tener claro [[KUBERNETES|Kubernetes en general]] primero — esta nota es específicamente lo que Azure agrega/gestiona por vos.

## Qué gestiona Azure vs qué gestionás vos

| Componente | Quién lo gestiona en AKS |
|---|---|
| Control Plane (API Server, etcd, Scheduler) | **Azure** — gratis, sin acceso directo, alta disponibilidad incluida |
| Nodos worker (VMs) | **Vos** (Azure los provisiona, pero decidís tamaño, cantidad, autoscaling) |
| Networking, storage, identidad | integrados con el resto de Azure (VNets, Managed Disks, Azure AD) |

Esto es la ventaja central de un Kubernetes gestionado: no administrás el Control Plane (parches de seguridad, HA de etcd, etc.) — Azure se encarga, vos te enfocás en los nodos y las apps.

## Crear un cluster AKS (CLI)

```bash
az group create --name mi-rg --location eastus

az aks create \
  --resource-group mi-rg \
  --name mi-cluster-aks \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --enable-managed-identity \
  --generate-ssh-keys

az aks get-credentials --resource-group mi-rg --name mi-cluster-aks
kubectl get nodes
```

`az aks get-credentials` configura el `kubeconfig` local para que `kubectl` apunte al cluster de Azure — a partir de ahí, todo lo que ya sabés de Kubernetes "puro" aplica igual.

## Node Pools: distintos tipos de nodos en el mismo cluster

```bash
az aks nodepool add \
  --resource-group mi-rg \
  --cluster-name mi-cluster-aks \
  --name gpupool \
  --node-vm-size Standard_NC6 \
  --node-count 1
```

Útil para separar cargas de trabajo (ej. un pool para apps normales, otro con GPU para ML, otro con spot instances para jobs tolerantes a interrupción).

## Cluster Autoscaler y Node Autoprovisioning

```bash
az aks nodepool update \
  --resource-group mi-rg \
  --cluster-name mi-cluster-aks \
  --name nodepool1 \
  --enable-cluster-autoscaler \
  --min-count 2 \
  --max-count 6
```

Se combina con el **Horizontal Pod Autoscaler** (HPA, a nivel Kubernetes): HPA escala Pods según CPU/memoria; si ya no entran en los nodos actuales, el Cluster Autoscaler agrega nodos nuevos automáticamente.

## Integración con Azure AD (RBAC)

```bash
az aks create \
  --resource-group mi-rg \
  --name mi-cluster-aks \
  --enable-aad \
  --enable-azure-rbac
```

Con esto, el acceso al cluster (quién puede hacer qué vía `kubectl`) se gestiona con roles de Azure AD en vez de certificados/usuarios separados de Kubernetes — ver [[06 - Namespaces RBAC y Seguridad Basica]] para el RBAC nativo de K8s que esto complementa.

## Managed Identity: acceso a otros recursos de Azure sin secrets

```bash
az aks create --resource-group mi-rg --name mi-cluster-aks --enable-managed-identity
```

Con Workload Identity (evolución de pod-managed identity), un Pod puede autenticarse contra Azure Key Vault, Storage, etc. sin necesitar credenciales guardadas — mismo principio que OIDC visto en GitHub Actions ([[04 - Secrets Variables y Environments]]), aplicado ahora a nivel Pod.

## ACR (Azure Container Registry): dónde viven las imágenes

```bash
az acr create --resource-group mi-rg --name miacr --sku Basic
az aks update --resource-group mi-rg --name mi-cluster-aks --attach-acr miacr
```

`--attach-acr` le da al cluster permiso para pullear imágenes del registry privado sin configurar `imagePullSecrets` manualmente.

```yaml
spec:
  containers:
    - name: app
      image: miacr.azurecr.io/mi-app:1.0
```

## Deploy típico a AKS desde CI/CD

```
build → push a ACR → kubectl apply / helm upgrade contra el cluster AKS
```

Esto conecta directo con lo que ya viste en [[GitHub Actions CI CD]] (módulo de Deploy) — el mismo patrón de "build separado de deploy", ahora con ACR como registry y AKS como destino.

## Errores comunes

- Pensar que AKS es "gratis" porque el Control Plane lo es — los nodos worker (VMs) sí generan costo, y es el componente que hay que dimensionar con criterio.
- No usar Managed Identity y terminar guardando credenciales de Azure como Secrets de Kubernetes (contradice el punto de tener Azure AD integrado).
- Un solo Node Pool para todo tipo de carga, sin aprovechar node pools especializados o spot instances para jobs no críticos (impacto directo en costos).
- Olvidar `--attach-acr` y lidiar manualmente con `imagePullSecrets`.

## Checklist
- [ ] Entiendo qué gestiona Azure (Control Plane) vs qué gestiono yo (nodos) en AKS.
- [ ] Sé crear un cluster AKS básico por CLI y conectar kubectl.
- [ ] Entiendo Node Pools y cuándo separar cargas de trabajo.
- [ ] Sé cómo se combina Cluster Autoscaler (nodos) con HPA (Pods).
- [ ] Conozco Managed Identity/Workload Identity como alternativa a secrets estáticos.
- [ ] Sé integrar ACR con AKS para pullear imágenes privadas.
