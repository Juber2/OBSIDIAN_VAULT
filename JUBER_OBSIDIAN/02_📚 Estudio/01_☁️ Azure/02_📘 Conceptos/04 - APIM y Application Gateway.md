---
tags:
  - Azure
  - PaaS
Creado: 2026-08-12
Relacionado:
  - Azure
---

# APIM y Application Gateway (deseables del puesto)

## Azure API Management (APIM): gateway a nivel API

No es lo mismo que el Ingress de Kubernetes — APIM opera un nivel más arriba, como **gateway de gestión de APIs** para toda la organización (internas y externas), no solo para un cluster.

Funciones típicas:
- **Rate limiting / throttling**: limitar cuántas requests por minuto puede hacer un consumidor.
- **Autenticación centralizada**: validar API keys, JWT, OAuth2 en un solo punto, antes de llegar al backend real.
- **Transformación de requests/responses**: modificar headers, body, versión de la API sin tocar el backend.
- **Portal de desarrolladores**: documentación auto-publicada (basada en OpenAPI) para que consumidores externos se registren y prueben la API.
- **Versionado y revisiones**: exponer v1 y v2 de una API simultáneamente, con reglas de transición.

```
Cliente → APIM (auth, rate limit, transformación) → Backend real (App Service / AKS / Functions)
```

```bash
az apim create --name mi-apim --resource-group mi-rg --publisher-email admin@empresa.com --publisher-name "Mi Empresa" --sku-name Developer
```

Policy de ejemplo (XML, así se configuran en APIM):
```xml
<policies>
  <inbound>
    <rate-limit calls="100" renewal-period="60" />
    <validate-jwt header-name="Authorization">
      <openid-config url="https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration" />
    </validate-jwt>
  </inbound>
</policies>
```

## Application Gateway: load balancer L7 con WAF

Distinto de APIM (que gestiona APIs) — Application Gateway es un **balanceador de capa 7 (HTTP/HTTPS)** con:
- **WAF (Web Application Firewall)** integrado: bloquea patrones de ataque conocidos (SQLi, XSS) según reglas OWASP.
- **Enrutamiento por path/host**, similar a un Ingress.
- **SSL termination**: descifra HTTPS en el gateway, tráfico interno puede ir sin cifrar (o re-cifrado según política).
- **Autoscaling** gestionado por Azure.

Ya se mencionó en [[05 - Ingress y Exposicion Externa]]: AGIC (Application Gateway Ingress Controller) conecta este servicio directamente con Ingress de Kubernetes en AKS.

## Cuándo usar cada uno (y cuándo los tres juntos)

| Necesito... | Servicio |
|---|---|
| Gestionar y versionar APIs para consumidores externos, con portal de developers | APIM |
| Balancear tráfico HTTP con WAF hacia mis backends (AKS, App Service, VMs) | Application Gateway |
| Enrutar tráfico dentro del cluster Kubernetes | Ingress (NGINX o AGIC) |

Arquitectura típica combinada:
```
Internet → APIM (gestión, auth, rate limit) → Application Gateway (WAF, LB) → AKS Ingress → Services → Pods
```

No siempre se necesitan los tres — para un backend interno simple, un Ingress alcanza. APIM se justifica cuando hay consumidores externos (partners, apps móviles públicas) que necesitan un contrato de API gestionado.

## Errores comunes

- Confundir APIM con un simple reverse proxy — su valor real está en gestión de ciclo de vida de API, no solo en enrutar tráfico.
- No usar WAF en Application Gateway para APIs expuestas a internet, dejando el backend sin protección contra ataques comunes.
- Poner lógica de negocio en las policies de APIM (transformaciones complejas) en vez de mantenerla en el backend — APIM debería quedarse en el rol de gateway, no de lógica de aplicación.

## Checklist
- [ ] Puedo explicar la diferencia entre APIM, Application Gateway e Ingress de Kubernetes.
- [ ] Sé para qué sirve rate limiting y autenticación centralizada en un gateway.
- [ ] Entiendo qué es WAF y por qué importa para APIs expuestas a internet.
- [ ] Puedo describir una arquitectura combinada (APIM + App Gateway + AKS) y justificar cada capa.
