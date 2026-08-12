---
tags:
  - Kubernetes
Creado: 2026-08-09
Relacionado:
  - KUBERNETES
---

# Pods, Deployments y ReplicaSets

## Jerarquía

```
Deployment
   └── ReplicaSet (gestiona réplicas)
          └── Pod, Pod, Pod...
```

No se crean Pods sueltos en producción — se declara un **Deployment**, que crea y gestiona un **ReplicaSet**, que a su vez mantiene N Pods vivos.

## Pod manifest mínimo

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-app
spec:
  containers:
    - name: app
      image: mi-registry/mi-app:1.0
      ports:
        - containerPort: 8080
```

## Deployment (lo que realmente se usa)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mi-app
  template:
    metadata:
      labels:
        app: mi-app
    spec:
      containers:
        - name: app
          image: mi-registry/mi-app:1.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get replicasets
kubectl get pods -l app=mi-app
```

## `selector` y `labels`: cómo se conectan las piezas

El Deployment "encuentra" sus Pods por **labels**, no por nombre. `selector.matchLabels` debe coincidir con `template.metadata.labels`. Este mismo mecanismo de labels/selectors es el que después conecta Services con Pods (módulo 3).

## Rolling updates

Al cambiar la imagen de un Deployment, K8s hace rolling update por defecto: crea Pods nuevos, espera que estén listos, y recién ahí apaga los viejos — sin downtime.

```bash
kubectl set image deployment/mi-app app=mi-registry/mi-app:1.1
kubectl rollout status deployment/mi-app
kubectl rollout undo deployment/mi-app   # rollback si algo sale mal
```

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

## Resource requests y limits (por qué importan)

- `requests`: lo que el Pod necesita garantizado — el Scheduler usa esto para decidir en qué nodo entra.
- `limits`: el techo — si el contenedor lo supera en memoria, se lo mata (OOMKilled); en CPU, se lo throttlea.

Sin `requests`/`limits` definidos, un Pod puede acaparar recursos y afectar a otros en el mismo nodo — es una de las causas más comunes de incidentes en producción.

## Health checks: liveness y readiness probes

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

- **Liveness**: si falla, K8s reinicia el contenedor (está "colgado").
- **Readiness**: si falla, el Pod sale del pool de tráfico (Service) pero no se reinicia — útil mientras arranca o está sobrecargado momentáneamente.

## Errores comunes

- No poner `resources` → un Pod con memory leak puede tumbar el nodo entero.
- Confundir liveness con readiness — reiniciar un Pod que solo estaba "ocupado" empeora el problema.
- Editar Pods directamente en vez del Deployment — se pierden los cambios en el próximo reconcile.

## Checklist
- [ ] Entiendo la jerarquía Deployment → ReplicaSet → Pod.
- [ ] Sé escribir un Deployment con labels/selectors correctos.
- [ ] Configuro `requests` y `limits` siempre.
- [ ] Distingo liveness de readiness probes y sé cuándo usar cada una.
- [ ] Sé hacer y revertir un rolling update.
