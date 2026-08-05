---
tags:
  - Docker
  - DevOps
  - Docker-Orquestacion
Creado: 2026-07-28
Relacionado:
  - Orquestacion Basica
  - DOCKER
---

# 03 - Introduccion Conceptual a Kubernetes

## Que vas a aprender

En este capitulo vas a aprender los conceptos fundamentales de **Kubernetes** desde la perspectiva de alguien que ya conoce Docker: como se relacionan Pods, Deployments y Services con lo que ya conoces de containers y Compose, sin entrar en la operacion detallada del orquestador (que amerita su propio libro dedicado).

## Explicacion

Un **Pod** es la unidad minima de despliegue en Kubernetes, y tipicamente contiene un unico container de aplicacion (aunque puede tener mas de uno que comparten red y almacenamiento). Un **Deployment** es una capa encima de los Pods que gestiona replicas, actualizaciones rolling y rollback, conceptualmente similar a lo que un `service:` con `deploy:` hace en Swarm, pero con muchisimas mas opciones de configuracion. Un **Service** de Kubernetes (nombre que coincide con el termino pero es un concepto distinto al `services:` de Compose) es el mecanismo de red que da una direccion estable para alcanzar un grupo de Pods, sin importar cuales especificamente esten corriendo en un momento dado.

Kubernetes ademas introduce **namespaces** (aislamiento logico de recursos dentro del mismo cluster), **ConfigMaps y Secrets** (gestion de configuracion y secretos nativa), y un ecosistema enorme de herramientas y operadores de terceros que extienden sus capacidades para casi cualquier necesidad de infraestructura.

## Modelo mental

Pensa un Pod como el equivalente aproximado a un container corriendo (aunque tecnicamente puede contener mas de uno). Un Deployment es como la version mucho mas sofisticada de la seccion `deploy:` de Compose/Swarm que ya conoces, con mas perillas de control. Un Service de Kubernetes es como el DNS interno de una red custom de Docker (ver capitulo de Redes), pero con mucho mas poder de enrutamiento y balanceo.

## Ejemplo base

```yaml
# Deployment: analogo mas sofisticado de "service" con "deploy: replicas" en Swarm
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels: { app: api }
  template:
    metadata:
      labels: { app: api }
    spec:
      containers:
        - name: api
          image: mi-registro/api:1.4.0
          ports: [{ containerPort: 3000 }]
---
# Service: analogo mas sofisticado del DNS interno de una red custom de Docker
apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector: { app: api }
  ports: [{ port: 80, targetPort: 3000 }]
```

## Como se usa en un proyecto real

1. Se aprovecha el conocimiento ya adquirido de Docker (imagenes, capas, multi-stage builds) directamente en Kubernetes, dado que sigue usando el mismo formato de imagenes OCI.
2. Se traduce mentalmente cada concepto ya conocido (replicas de Compose/Swarm -> Deployment, red custom -> Service, `.env` -> ConfigMap/Secret) para acelerar el aprendizaje de Kubernetes.
3. Se reconoce que Kubernetes es significativamente mas complejo de operar directamente, y en la practica la mayoria de equipos usan un servicio gestionado (EKS, GKE, AKS) que abstrae buena parte de la administracion del cluster en si.
4. Se evalua adoptar Kubernetes solo cuando las necesidades reales (escala, ecosistema, equipos grandes con necesidades de aislamiento fuerte) superan claramente lo que Swarm o un enfoque mas simple pueden ofrecer.
5. Se profundiza en Kubernetes como un tema de estudio dedicado y extenso, dado que este tema es solo una introduccion conceptual, no una guia operativa completa.

## Criterio junior

Un junior debe poder relacionar los conceptos basicos de Kubernetes (Pod, Deployment, Service) con los equivalentes aproximados que ya conoce de Docker y Compose.

## Criterio mid

Un mid debe entender las diferencias reales de complejidad operativa entre Kubernetes y Swarm, y reconocer cuando un proyecto justifica esa complejidad adicional.

## Criterio senior

Un senior debe decidir con criterio informado si Kubernetes es la eleccion correcta para un sistema dado, considerando el ecosistema disponible (operadores, herramientas de observabilidad) frente al costo operativo, y liderar la decision de adoptar un servicio gestionado en vez de operar Kubernetes desde cero.

## Errores comunes

- Adoptar Kubernetes sin la experiencia o el equipo necesario para operarlo con seguridad, generando mas riesgo que el problema que se intentaba resolver.
- Confundir el `Service` de Kubernetes con el `services:` de Docker Compose, que son conceptos con el mismo nombre pero significados distintos.
- Subestimar la curva de aprendizaje real de Kubernetes al venir de Docker/Compose, asumiendo que la transicion sera trivial.
- No considerar servicios gestionados (EKS, GKE, AKS) que reducen significativamente la complejidad operativa de correr Kubernetes.

## Practica

- [ ] Relaciona cada concepto basico de Kubernetes visto aca con su equivalente aproximado en Docker/Compose/Swarm.
- [ ] Investiga la diferencia practica entre operar Kubernetes desde cero vs usar un servicio gestionado.
- [ ] Identifica si algun proyecto que conozcas realmente justificaria la complejidad de Kubernetes.
- [ ] Anota que temas de Kubernetes necesitarias profundizar antes de operarlo en produccion con confianza.
- [ ] Compara conceptualmente el ecosistema de operadores/herramientas de Kubernetes con las capacidades de Swarm.

## Preguntas de repaso

- Que relacion aproximada hay entre un Deployment de Kubernetes y la seccion `deploy:` de Swarm/Compose?
- Por que el `Service` de Kubernetes no es lo mismo que el `services:` de Docker Compose, pese al nombre compartido?
- Que ventaja da usar un servicio gestionado (EKS, GKE, AKS) sobre operar Kubernetes desde cero?
- Que factores determinarian si un proyecto realmente justifica la complejidad operativa de Kubernetes?
