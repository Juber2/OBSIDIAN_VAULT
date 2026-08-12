---
tags:
  - Kubernetes
Creado: 2026-08-12
Relacionado:
  - KUBERNETES
---

# Services y Networking

## El problema que resuelve Service

Los Pods son efímeros: mueren y se recrean con **IP nueva** cada vez. Si otro Pod se conecta directo a la IP de un Pod, se rompe apenas ese Pod se reinicia. Un **Service** da una IP/DNS estable que enruta tráfico hacia el set de Pods correcto (vía labels), sin importar que los Pods de atrás cambien.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-app-svc
spec:
  selector:
    app: mi-app          # mismo label que el Deployment
  ports:
    - port: 80            # puerto del Service
      targetPort: 8080     # puerto del contenedor
```

```bash
kubectl get svc
kubectl get endpoints mi-app-svc   # ver qué Pods está enrutando de verdad
```

## Tipos de Service

| Tipo | Alcance | Cuándo usarlo |
|---|---|---|
| `ClusterIP` (default) | solo dentro del cluster | comunicación interna entre microservicios |
| `NodePort` | expone un puerto fijo en cada nodo | pruebas rápidas, pocas veces en producción |
| `LoadBalancer` | provisiona un balanceador externo (cloud) | exponer un servicio a internet en AKS/EKS/GKE |
| `ExternalName` | mapea a un DNS externo | apuntar a un servicio fuera del cluster |

```yaml
spec:
  type: LoadBalancer
  selector:
    app: mi-app
  ports:
    - port: 80
      targetPort: 8080
```

En AKS, `type: LoadBalancer` provisiona automáticamente un Azure Load Balancer con IP pública — no hace falta configurarlo manualmente por fuera.

## DNS interno: cómo se llaman los servicios entre sí

Kubernetes tiene DNS interno (CoreDNS). Cualquier Pod puede resolver un Service por nombre:

```
http://mi-app-svc                                  # mismo namespace
http://mi-app-svc.otro-namespace.svc.cluster.local  # namespace distinto, forma completa
```

Esto es lo que reemplaza a "hardcodear IPs" o usar service discovery externo (como Eureka en Spring Cloud) — Kubernetes ya lo resuelve a nivel de plataforma.

## Comunicación Pod a Pod dentro del cluster

Cada Pod tiene su propia IP, y por default **todos los Pods pueden hablarse entre sí** sin NAT (modelo de red plano de Kubernetes) — a menos que se restrinja explícitamente con Network Policies.

## Network Policies: restringir tráfico

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: solo-desde-api
spec:
  podSelector:
    matchLabels:
      app: base-datos
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api
      ports:
        - port: 5432
```

Sin Network Policy, cualquier Pod puede conectarse a cualquier otro — en producción esto es un riesgo de seguridad. Esta policy dice "solo Pods con label `app: api` pueden conectarse a `base-datos` por el puerto 5432".

## Errores comunes

- Confundir `port` (puerto del Service) con `targetPort` (puerto real del contenedor) — si no coinciden con lo que expone la app, el tráfico no llega.
- Usar `NodePort` en producción por simplicidad, exponiendo puertos innecesarios en cada nodo.
- No usar Network Policies y asumir que "está dentro del cluster" es sinónimo de "seguro".
- Selector del Service que no coincide con los labels del Deployment → Service sin Endpoints, tráfico se pierde silenciosamente (siempre verificar con `kubectl get endpoints`).

## Checklist
- [ ] Entiendo por qué los Pods necesitan un Service delante (IPs efímeras).
- [ ] Sé elegir el tipo correcto de Service según el caso (ClusterIP vs LoadBalancer vs NodePort).
- [ ] Sé resolver un Service por DNS interno desde otro Pod.
- [ ] Sé escribir una Network Policy básica para restringir tráfico.
- [ ] Uso `kubectl get endpoints` para diagnosticar Services sin tráfico.
