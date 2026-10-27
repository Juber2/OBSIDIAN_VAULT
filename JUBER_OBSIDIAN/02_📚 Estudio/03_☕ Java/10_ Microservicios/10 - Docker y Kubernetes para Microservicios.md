---
tags:
  - Java
  - Microservicios
  - Docker
  - Kubernetes
Creado: 2026-05-21
Relacionado:
  - Microservicios
  - DevOps
Palabras clave:
  - Docker, Dockerfile, docker-compose, Kubernetes, K8s, deployment, service
---

## Dockerfile para microservicio Java

```dockerfile
# Etapa 1: Build
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY pom.xml .
COPY .mvn .mvn
COPY mvnw .
# Descargar dependencias (cacheado si pom.xml no cambia)
RUN ./mvnw dependency:resolve -q

COPY src src
RUN ./mvnw package -DskipTests -q

# Etapa 2: Runtime (imagen mínima)
FROM eclipse-temurin:21-jre-alpine AS runtime
WORKDIR /app

# Usuario no-root por seguridad
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080

# JVM tuning para contenedores
ENTRYPOINT ["java",
    "-XX:+UseContainerSupport",
    "-XX:MaxRAMPercentage=75.0",
    "-XX:+UseG1GC",
    "-Djava.security.egd=file:/dev/./urandom",
    "-jar", "app.jar"]
```

## docker-compose para desarrollo local

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Infraestructura
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: orders_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
    depends_on:
      - zookeeper

  zookeeper:
    image: confluentinc/cp-zookeeper:7.6.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  zipkin:
    image: openzipkin/zipkin:latest
    ports:
      - "9411:9411"

  # Servicios de la aplicación
  eureka-server:
    build: ./eureka-server
    ports:
      - "8761:8761"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8761/actuator/health"]
      interval: 10s
      timeout: 5s
      retries: 5

  config-server:
    build: ./config-server
    ports:
      - "8888:8888"
    depends_on:
      eureka-server:
        condition: service_healthy
    environment:
      EUREKA_URI: http://eureka-server:8761/eureka

  api-gateway:
    build: ./api-gateway
    ports:
      - "8080:8080"
    depends_on:
      - eureka-server
      - config-server
    environment:
      EUREKA_URI: http://eureka-server:8761/eureka
      CONFIG_URI: http://config-server:8888

  order-service:
    build: ./order-service
    ports:
      - "8082:8082"
    depends_on:
      postgres:
        condition: service_healthy
      kafka:
        condition: service_started
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/orders_db
      SPRING_KAFKA_BOOTSTRAP_SERVERS: kafka:9092
      EUREKA_URI: http://eureka-server:8761/eureka
    deploy:
      replicas: 2   # 2 instancias del servicio

volumes:
  postgres_data:
```

## Kubernetes — desplegar microservicio

```yaml
# order-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: microservicios
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
        - name: order-service
          image: empresa/order-service:1.0.0
          ports:
            - containerPort: 8082
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: order-service-secrets
                  key: db-password
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8082
            initialDelaySeconds: 30
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8082
            initialDelaySeconds: 60
            periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: order-service
  namespace: microservicios
spec:
  selector:
    app: order-service
  ports:
    - port: 80
      targetPort: 8082
  type: ClusterIP   # solo dentro del cluster
```

## Spring Boot + K8s — Actuator Probes

```yaml
# application.yml
management:
  endpoint:
    health:
      probes:
        enabled: true
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
```

```bash
# Endpoints automáticos:
/actuator/health/liveness   → está vivo el proceso?
/actuator/health/readiness  → está listo para recibir tráfico?
```

## ConfigMap y Secret

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: order-service-config
data:
  application.yml: |
    app:
      orders:
        max-items: 50
    logging:
      level:
        root: INFO

---
# secret.yaml (valores en base64)
apiVersion: v1
kind: Secret
metadata:
  name: order-service-secrets
type: Opaque
data:
  db-password: cGFzc3dvcmQxMjM=   # base64("password123")
  jwt-secret: bXktc2VjcmV0LWtleQ==
```

## Horizontal Pod Autoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70  # escalar si CPU > 70%
```

## 📌 Notas Rápidas
- El Dockerfile multi-stage (build en una imagen JDK, runtime en una JRE Alpine) es exactamente el mismo patrón que se usa para Quarkus native (ver [[08 - Native Image con GraalVM]]) — imagen final más chica y sin herramientas de build innecesarias en producción.
- `-XX:+UseContainerSupport` y `-XX:MaxRAMPercentage` son claves en contenedores: sin esto, la JVM históricamente no detectaba bien los límites de memoria del contenedor (cgroups) y podía sobreasignar heap y ser OOMKilled por Kubernetes.
- `readinessProbe`/`livenessProbe` apuntando a Actuator (`/actuator/health/readiness`, `/actuator/health/liveness`) es la integración directa entre Spring Boot y el mecanismo de salud de Kubernetes visto en [[02 - Pods Deployments y ReplicaSets]].
- El HPA (Horizontal Pod Autoscaler) escala Pods según CPU/memoria; combinado con el Cluster Autoscaler de AKS (ver [[02 - AKS Fundamentos]]) se obtiene autoscaling de extremo a extremo: más carga → más Pods → más nodos si hace falta.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

