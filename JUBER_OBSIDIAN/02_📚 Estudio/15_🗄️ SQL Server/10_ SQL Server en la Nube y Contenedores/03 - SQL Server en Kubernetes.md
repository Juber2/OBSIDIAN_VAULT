---
tags:
  - SQLServer
  - Nube
  - SQLServer-Nube
Creado: 2026-07-29
Relacionado:
  - SQL Server en la Nube y Contenedores
  - SQL SERVER
---

# 03 - SQL Server en Kubernetes

## Que vas a aprender

En este capitulo vas a aprender a correr SQL Server en Kubernetes usando `StatefulSet` y `PersistentVolumeClaim`, aplicando directamente los mismos conceptos ya estudiados para bases de datos con estado en Kubernetes en general, mas el operador especifico de Microsoft (`SQL Server on Kubernetes` / Big Data Cluster, segun la version).

## Explicacion

Igual que cualquier base de datos con estado en Kubernetes, SQL Server necesita un `StatefulSet` (que garantiza identidad de red estable y volumen persistente asociado a cada replica, a diferencia de un `Deployment` comun) respaldado por un `PersistentVolumeClaim` para que los archivos `.mdf`/`.ldf` sobrevivan al reinicio o recreacion del Pod. Microsoft tambien ofrece el **SQL Server Kubernetes operator**, que automatiza el despliegue de Availability Groups completos dentro de Kubernetes (gestionando el failover automatico como recursos nativos de Kubernetes), reduciendo la complejidad manual de configurar Always On AG sobre infraestructura tradicional.

Para cargas de produccion serias con alta disponibilidad, correr Always On AG dentro de Kubernetes vía el operador oficial es la opcion moderna recomendada frente a configurar manualmente un Windows Server Failover Cluster tradicional, de forma analoga a como Patroni automatiza HA de Postgres en Kubernetes (ya visto en ese libro).

## Modelo mental

Pensa SQL Server en Kubernetes exactamente igual que Postgres en Kubernetes ya estudiado: un `StatefulSet` le da identidad estable y almacenamiento persistente a cada replica, y un operador especializado (Patroni para Postgres, el SQL Server operator para SQL Server) automatiza la logica compleja de failover que de otro modo tendrias que orquestar manualmente vos mismo.

## Ejemplo base

```yaml
# StatefulSet simplificado para SQL Server en Kubernetes
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sqlserver
spec:
  serviceName: sqlserver
  replicas: 1
  selector:
    matchLabels:
      app: sqlserver
  template:
    metadata:
      labels:
        app: sqlserver
    spec:
      containers:
        - name: sqlserver
          image: mcr.microsoft.com/mssql/server:2022-latest
          env:
            - name: ACCEPT_EULA
              value: "Y"
            - name: MSSQL_SA_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: sqlserver-secret
                  key: sa-password
          volumeMounts:
            - name: data
              mountPath: /var/opt/mssql
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 20Gi
```

## Como se usa en un proyecto real

1. Se usa un `StatefulSet` con `PersistentVolumeClaim` para correr SQL Server en un cluster de Kubernetes, garantizando que los datos sobrevivan al reinicio o recreacion del Pod.
2. Se evalua el SQL Server Kubernetes operator para desplegar Always On AG completo dentro de Kubernetes cuando el sistema requiere alta disponibilidad real, evitando la complejidad manual de un cluster tradicional.
3. Se define la contraseña de `sa` como un `Secret` de Kubernetes, nunca como variable de entorno en texto plano dentro del manifiesto.
4. Se conecta esto directamente con Postgres en Kubernetes (StatefulSet + Patroni) ya estudiado en ese libro, aplicando el mismo modelo mental.
5. Se dimensiona correctamente el `PersistentVolumeClaim` y los recursos de CPU/memoria del Pod, dado que SQL Server tiene requisitos minimos mas altos que Postgres tipicamente.

## Criterio junior

Un junior debe poder desplegar SQL Server en Kubernetes usando un `StatefulSet` con `PersistentVolumeClaim` basico.

## Criterio mid

Un mid debe usar `Secret` para credenciales y dimensionar apropiadamente los recursos del Pod.

## Criterio senior

Un senior debe evaluar y desplegar el SQL Server Kubernetes operator para Always On AG en produccion, comparandolo con alternativas administradas como Azure SQL Database.

## Errores comunes

- Usar un `Deployment` en vez de `StatefulSet`, perdiendo la identidad de red estable y garantia de volumen persistente que SQL Server necesita.
- Definir la contraseña de `sa` como variable de entorno en texto plano dentro del manifiesto en vez de usar un `Secret`.
- No dimensionar correctamente los recursos del Pod, causando OOMKill o performance degradada por requisitos minimos no cubiertos.
- Configurar Always On AG manualmente dentro de Kubernetes sin usar el operador oficial, reinventando logica de failover ya resuelta.

## Practica

- [ ] Despliega SQL Server en un cluster de Kubernetes local (Minikube o Kind) usando un `StatefulSet`.
- [ ] Configura la contraseña de `sa` como un `Secret` de Kubernetes.
- [ ] Investiga el SQL Server Kubernetes operator y como automatiza Always On AG.
- [ ] Compara este despliegue con Postgres en Kubernetes usando Patroni, ya estudiado en ese libro.
- [ ] Dimensiona los recursos de CPU/memoria apropiados para una carga de produccion hipotetica.

## Preguntas de repaso

- Por que se usa `StatefulSet` en vez de `Deployment` para SQL Server en Kubernetes?
- Que automatiza el SQL Server Kubernetes operator?
- Como se compara este enfoque con Patroni para Postgres en Kubernetes?
- Que consideraciones de dimensionamiento son especificas de SQL Server frente a Postgres en un Pod?
