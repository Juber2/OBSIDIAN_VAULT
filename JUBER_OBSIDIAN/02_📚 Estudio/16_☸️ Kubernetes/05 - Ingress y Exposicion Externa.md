---
tags:
  - Kubernetes
Creado: 2026-08-12
Relacionado:
  - KUBERNETES
---

# Ingress y Exposición Externa

## El problema: un LoadBalancer por Service es caro y desordenado

Con `type: LoadBalancer` cada Service crea su propio balanceador cloud (costo por cada uno). Si tenés 10 microservicios expuestos, son 10 LoadBalancers. **Ingress** resuelve esto: un solo punto de entrada HTTP/HTTPS que enruta por dominio/path hacia los Services internos (que quedan como `ClusterIP`).

```
Internet → Ingress Controller (1 LoadBalancer) → Ingress rules → Service A / Service B / Service C
```

## Ingress Controller vs Ingress resource

- **Ingress Controller**: el software que efectivamente hace el enrutamiento (NGINX Ingress, Azure Application Gateway Ingress Controller / AGIC, Traefik). No viene instalado por default en el cluster — hay que desplegarlo.
- **Ingress**: el recurso YAML que declara las reglas de enrutamiento, que el Controller lee y aplica.

## Ingress básico

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mi-app-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: api.miapp.com
      http:
        paths:
          - path: /productos
            pathType: Prefix
            backend:
              service:
                name: producto-service
                port:
                  number: 80
          - path: /pedidos
            pathType: Prefix
            backend:
              service:
                name: pedido-service
                port:
                  number: 80
```

Un solo dominio (`api.miapp.com`), rutea `/productos` a un microservicio y `/pedidos` a otro — patrón típico de exponer varios microservicios detrás de un único gateway.

## TLS/HTTPS

```yaml
spec:
  tls:
    - hosts:
        - api.miapp.com
      secretName: api-miapp-tls   # Secret tipo kubernetes.io/tls con cert + key
  rules:
    - host: api.miapp.com
      # ...
```

En producción esto se combina con **cert-manager**, que automatiza la emisión/renovación de certificados (ej. vía Let's Encrypt), en vez de gestionar certificados a mano.

## AGIC: Azure Application Gateway como Ingress Controller

En AKS, en vez de usar NGINX Ingress genérico, se puede usar **AGIC (Application Gateway Ingress Controller)** — integra el Ingress de Kubernetes directamente con Azure Application Gateway (un servicio PaaS de Azure), ganando WAF (Web Application Firewall) integrado, autoscaling del gateway gestionado por Azure, y mejor integración con el resto de la red virtual de Azure. Es la opción "cloud native" cuando el cluster corre en AKS, en vez de reinventar lo que Azure ya ofrece como servicio administrado.

## Errores comunes

- Desplegar un Ingress sin tener un Ingress Controller corriendo en el cluster — el recurso queda sin efecto, nada enruta.
- Exponer cada microservicio con su propio `LoadBalancer` en vez de centralizar con Ingress — más costo, más superficie de ataque, más difícil de gestionar TLS de forma consistente.
- No configurar TLS y exponer APIs por HTTP plano a internet.
- No usar `pathType: Prefix` correctamente y terminar con rutas que no matchean como se espera (revisar `Exact` vs `Prefix` vs `ImplementationSpecific`).

## Checklist
- [ ] Entiendo la diferencia entre Ingress Controller (el software) e Ingress (la regla).
- [ ] Sé enrutar múltiples microservicios detrás de un solo dominio con paths distintos.
- [ ] Sé configurar TLS en un Ingress.
- [ ] Conozco AGIC como opción nativa de Azure para AKS y por qué se preferiría sobre NGINX genérico en ese contexto.
