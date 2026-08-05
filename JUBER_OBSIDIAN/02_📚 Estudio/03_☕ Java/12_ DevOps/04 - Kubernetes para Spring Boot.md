---
tags:
  - Java
  - Kubernetes
Creado: 2026-05-21
Relacionado:
  - DevOps con Java
---

# Kubernetes para Spring Boot

Kubernetes orquesta contenedores. Para Spring Boot necesitas entender deployment, service, configmaps, secrets, probes y recursos.

## Deployment basico

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: user-api
  template:
    metadata:
      labels:
        app: user-api
    spec:
      containers:
        - name: user-api
          image: user-api:1.0.0
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
```

## Conceptos clave

| Concepto | Uso |
|---|---|
| Deployment | Version y replicas |
| Service | Red estable interna |
| ConfigMap | Configuracion no secreta |
| Secret | Valores sensibles |
| Probes | Saber si la app vive y esta lista |

## Checklist

- [ ] Defino requests y limits de CPU/memoria.
- [ ] Uso readiness y liveness probes.
- [ ] No guardo secretos en imagenes.
- [ ] La app apaga correctamente con graceful shutdown.
