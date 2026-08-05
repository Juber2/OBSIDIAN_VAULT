---
tags:
  - PostgreSQL
  - Cloud
  - Postgres-Cloud
Creado: 2026-07-29
Relacionado:
  - Postgres en la Nube y Contenedores
  - POSTGRESQL
---

# 03 - Postgres en Kubernetes

## Que vas a aprender

En este capitulo vas a aprender los desafios especificos de correr Postgres en **Kubernetes**, conectando con la introduccion conceptual de Kubernetes ya vista en el libro de Docker, y por que las bases de datos con estado necesitan consideraciones especiales en un orquestador diseñado originalmente para cargas sin estado.

## Explicacion

Kubernetes fue diseñado pensando primero en aplicaciones **stateless** (sin estado, como una API que puede recrearse en cualquier nodo sin perder nada, ver capitulo de Escalabilidad del libro de NestJS). Una base de datos es fundamentalmente **stateful**: no se puede simplemente recrear un Pod de Postgres en otro nodo sin preservar cuidadosamente sus datos. Kubernetes resuelve esto con **StatefulSets** (en vez de Deployments normales) y **PersistentVolumes** (el equivalente de Kubernetes a los volumenes de Docker ya estudiados), garantizando identidad estable y almacenamiento persistente correctamente asociado a cada replica de Postgres.

Dado que Patroni (ya visto en el capitulo de Alta Disponibilidad) ya resuelve el problema de failover automatico, la practica estandar en Kubernetes es usar un **operador** especializado (como el Zalando Postgres Operator, o CloudNativePG) que combina Patroni con la logica de Kubernetes para automatizar completamente el ciclo de vida de un cluster Postgres: creacion, backups, escalado de replicas, y failover, todo declarado como configuracion de Kubernetes.

## Modelo mental

Pensa correr Postgres en Kubernetes sin un operador especializado como intentar mudar una casa entera (con todos sus muebles en el lugar exacto) usando un sistema de mudanzas diseñado para cajas genericas intercambiables: funciona, pero requiere mucho cuidado adicional. Un operador de Postgres es como contratar una empresa de mudanzas especializada en casas completas, que ya sabe exactamente como preservar cada mueble en su lugar durante todo el proceso, automatizando la complejidad especifica.

## Ejemplo base

```yaml
# Ejemplo conceptual usando CloudNativePG operator
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: mi-cluster-postgres
spec:
  instances: 3   # primario + 2 replicas, gestionado automaticamente
  storage:
    size: 20Gi
  backup:
    barmanObjectStore:
      destinationPath: s3://mi-bucket/backups
```

```bash
kubectl get clusters   # ver el estado del cluster gestionado por el operador
kubectl cnpg status mi-cluster-postgres   # detalle de primario/replicas/backups
```

## Como se usa en un proyecto real

1. Se usa un operador especializado (CloudNativePG, Zalando Postgres Operator) en vez de intentar administrar Postgres en Kubernetes manualmente con recursos genericos.
2. Se declara la configuracion del cluster Postgres (numero de replicas, backups, recursos) como YAML versionado, aplicando el mismo principio de infraestructura como codigo ya visto en el libro de Linux.
3. Se evalua correr Postgres en Kubernetes solo cuando el resto de la infraestructura ya vive ahi y el equipo tiene experiencia operando StatefulSets con datos criticos.
4. Se considera un servicio gestionado externo (ver tema anterior) incluso para aplicaciones que corren en Kubernetes, separando la base de datos de la complejidad adicional del orquestador cuando no aporta beneficio claro.
5. Se conecta esto directamente con la introduccion conceptual de Kubernetes del libro de Docker, aplicando esos conceptos basicos (Pods, PersistentVolumes) al caso especifico de una carga con estado.

## Criterio junior

Un junior debe entender por que una base de datos necesita StatefulSets y PersistentVolumes en vez de un Deployment normal en Kubernetes.

## Criterio mid

Un mid debe usar un operador especializado para desplegar un cluster Postgres basico en Kubernetes, declarando su configuracion como YAML.

## Criterio senior

Un senior debe decidir cuando correr Postgres en Kubernetes se justifica versus usar un servicio gestionado externo, considerando la experiencia operativa del equipo y la complejidad adicional que un operador todavia introduce.

## Errores comunes

- Intentar correr Postgres en Kubernetes con un Deployment normal en vez de un StatefulSet, sin entender por que el estado necesita ese tratamiento especial.
- No usar un operador especializado, intentando reimplementar manualmente logica de failover y backup que Patroni y los operadores ya resuelven de forma probada.
- Correr Postgres en Kubernetes sin experiencia real del equipo, subestimando la complejidad operativa adicional que un orquestador agrega sobre la ya presente en administrar Postgres.
- No evaluar seriamente un servicio gestionado externo como alternativa mas simple, incluso cuando el resto de la aplicacion ya vive en Kubernetes.

## Practica

- [ ] Investiga la diferencia entre un Deployment y un StatefulSet de Kubernetes para el caso especifico de una base de datos.
- [ ] Investiga que operadores de Postgres para Kubernetes existen (CloudNativePG, Zalando) y compara sus features.
- [ ] Si tenes acceso a un cluster Kubernetes de practica, despliega un cluster Postgres basico con un operador.
- [ ] Evalua si un proyecto real tuyo (o hipotetico) justificaria correr Postgres en Kubernetes versus un servicio gestionado externo.
- [ ] Conecta este tema con la introduccion de Kubernetes ya vista en el libro de Docker.

## Preguntas de repaso

- Por que una base de datos necesita un StatefulSet en vez de un Deployment normal en Kubernetes?
- Que ventaja da usar un operador especializado en vez de administrar Postgres manualmente en Kubernetes?
- Que experiencia operativa necesitaria un equipo antes de correr Postgres en Kubernetes con confianza?
- Cuando seguiria siendo mejor opcion un servicio gestionado externo aunque el resto de la aplicacion viva en Kubernetes?
