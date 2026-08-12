---
tags:
  - Kubernetes
  - Seguridad
Creado: 2026-08-12
Relacionado:
  - KUBERNETES
---

# Namespaces, RBAC y Seguridad Básica

## Namespaces: aislamiento lógico dentro del mismo cluster

Un cluster se divide en namespaces para separar entornos o equipos sin necesitar clusters físicos distintos.

```bash
kubectl create namespace staging
kubectl get pods -n staging
kubectl config set-context --current --namespace=staging   # cambiar el default
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: produccion
```

Patrón común: un namespace por entorno (`dev`, `staging`, `prod`) o por equipo/dominio. Los nombres de recursos solo necesitan ser únicos **dentro** del mismo namespace — `mi-app-svc` puede existir en `dev` y en `prod` sin conflicto.

## Resource Quotas: limitar consumo por namespace

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-staging
  namespace: staging
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```

Evita que un namespace (ej. un equipo con bugs de memoria) consuma todos los recursos del cluster y afecte a otros.

## RBAC: quién puede hacer qué

RBAC (Role-Based Access Control) controla permisos sobre la API de Kubernetes — no es autenticación de la app, es **quién puede usar `kubectl`/la API** y sobre qué recursos.

### Role (permisos dentro de un namespace)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: staging
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

### RoleBinding (asignar el Role a un usuario/grupo/service account)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: leer-pods-staging
  namespace: staging
subjects:
  - kind: User
    name: juber.gutierrez@empresa.com
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRole / ClusterRoleBinding: lo mismo pero a nivel cluster completo (no un namespace)

Útil para permisos que cruzan namespaces, o recursos que no son namespaced (ej. Nodes).

## Service Accounts: identidad para Pods, no para personas

Cada Pod corre con una Service Account (por default, una genérica del namespace). Si un Pod necesita llamar a la API de Kubernetes (ej. un controller custom), se le asigna una Service Account con permisos acotados vía RBAC — nunca darle permisos de admin por comodidad.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mi-app-sa
  namespace: staging
```

```yaml
spec:
  serviceAccountName: mi-app-sa
  containers:
    - name: app
      image: mi-app:1.0
```

## Integración con Azure AD (en AKS)

AKS permite integrar RBAC de Kubernetes con **Azure Active Directory**: los usuarios se autentican con su identidad de Azure AD, y los permisos sobre el cluster se gestionan igual que cualquier otro recurso de Azure (grupos, roles) en vez de gestionar usuarios de Kubernetes por separado. Esto es lo que en el mundo Azure Cloud Native se espera como práctica estándar, no RBAC aislado del resto de la identidad corporativa.

## Seguridad adicional a nivel Pod

```yaml
spec:
  containers:
    - name: app
      securityContext:
        runAsNonRoot: true
        readOnlyRootFilesystem: true
        allowPrivilegeEscalation: false
```

Mismo principio que en Docker (usuario no-root) — evita que, si el contenedor es comprometido, el atacante tenga privilegios de root dentro del nodo.

## Errores comunes

- Correr todo en el namespace `default` sin separar entornos ni equipos.
- Dar `cluster-admin` a Service Accounts de aplicaciones "porque es más fácil" — principio de menor privilegio roto.
- No usar ResourceQuotas y descubrir en producción que un namespace acaparó todo el cluster.
- Confundir RBAC de Kubernetes (quién administra el cluster) con autenticación/autorización de la aplicación (JWT, OAuth2) — son capas completamente distintas.

## Checklist
- [ ] Sé crear namespaces y moverme entre ellos con kubectl.
- [ ] Sé escribir un Role y un RoleBinding básicos.
- [ ] Entiendo la diferencia entre Role/RoleBinding (namespace) y ClusterRole/ClusterRoleBinding (cluster).
- [ ] Sé para qué sirve una Service Account y cuándo un Pod necesita una específica.
- [ ] Conozco la integración de AKS con Azure AD para RBAC.
- [ ] Aplico `securityContext` básico (non-root, sin privilege escalation) en mis manifests.
