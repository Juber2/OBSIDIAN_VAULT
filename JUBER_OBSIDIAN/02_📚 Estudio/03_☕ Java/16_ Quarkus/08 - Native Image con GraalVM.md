---
tags:
  - Java
  - Quarkus
  - GraalVM
Creado: 2026-08-09
Relacionado:
  - Quarkus
---

# Native Image con GraalVM

Esta es la característica insignia de Quarkus: compilar la app a un **binario nativo ejecutable**, sin JVM, con arranque en milisegundos.

## Cómo funciona (idea general)

GraalVM's `native-image` analiza el bytecode de toda la app (closed-world assumption: todo lo que se ejecuta debe ser alcanzable estáticamente desde el punto de entrada) y genera un ejecutable nativo del SO, con su propio runtime mínimo embebido (no hay JVM corriendo por debajo).

```
JVM mode:   java -jar app.jar        → arranca JVM, carga clases, ejecuta      (~1-3s)
Native:     ./app-runner             → binario nativo, ejecuta directo        (~10-50ms)
```

## Compilar una imagen nativa

```bash
./mvnw package -Dnative
```

Requiere GraalVM instalado localmente, **o** usar el contenedor de build (recomendado, evita instalar GraalVM en la máquina):

```bash
./mvnw package -Dnative -Dquarkus.native.container-build=true
```

Esto genera un ejecutable en `target/mi-servicio-1.0-runner`.

## Trade-offs reales (no es gratis)

| Aspecto | JVM mode | Native mode |
|---|---|---|
| Tiempo de build | segundos | minutos (varios, puede ser 3-10 min) |
| Arranque | ~1-3s | ~10-50ms |
| Memoria en runtime | mayor | menor |
| Peak throughput | generalmente mejor (JIT optimiza con el tiempo) | algo menor (sin JIT adaptativo en runtime) |
| Debugging | tooling JVM completo (profilers, etc.) | limitado |
| Reflection dinámica | funciona sin restricciones | requiere configuración explícita (ver abajo) |

**Conclusión práctica**: native conviene cuando el arranque rápido es la prioridad (K8s con autoscaling agresivo, serverless). Para un servicio de vida larga con tráfico estable, JVM mode con throughput optimizado por JIT puede ser mejor opción.

## El problema de la reflection

Como GraalVM analiza estáticamente qué código es alcanzable, cualquier uso de reflection **dinámica** (que no puede inferirse en build-time, ej. `Class.forName(variable)`) puede fallar en el binario nativo aunque funcione perfecto en JVM mode.

Solución: declarar explícitamente qué clases necesitan reflection.

```java
@RegisterForReflection
public class ProductoDTO {
    // ...
}
```

O vía configuración para clases de librerías de terceros:
```properties
quarkus.native.additional-build-args=-H:ReflectionConfigurationFiles=reflection-config.json
```

La mayoría de las extensions oficiales de Quarkus ya manejan esto internamente — el problema aparece con librerías genéricas no pensadas para compilación nativa.

## Dockerfile para native (multi-stage)

```dockerfile
FROM quay.io/quarkus/ubi-quarkus-native-image:22.3-java17 AS build
COPY . /project
WORKDIR /project
RUN ./mvnw package -Dnative -Dquarkus.native.container-build=true

FROM quay.io/quarkus/quarkus-micro-image:2.0
COPY --from=build /project/target/*-runner /work/application
WORKDIR /work
EXPOSE 8080
ENTRYPOINT ["./application"]
```

La imagen final resultante es drásticamente más chica y liviana que una imagen con JVM completa — impacto directo en costos de storage/transfer y en tiempos de pull en Kubernetes.

## Cuándo NO vale la pena compilar nativo

- Fase de desarrollo activo (el build nativo tarda minutos, rompe el feedback loop) — usar JVM mode o Dev Mode.
- Apps con dependencias pesadas en reflection dinámica no soportada, donde el esfuerzo de configurar excepciones supera el beneficio.
- Servicios donde el arranque no es un factor relevante (no escalan horizontalmente seguido).

## Checklist
- [ ] Entiendo qué hace `native-image` (closed-world, análisis estático).
- [ ] Sé compilar una imagen nativa con el contenedor de build.
- [ ] Sé cuándo un error es por reflection no declarada y cómo resolverlo con `@RegisterForReflection`.
- [ ] Puedo armar un Dockerfile multi-stage para native.
- [ ] Tengo criterio para decidir JVM mode vs Native mode según el caso real (no por moda).
