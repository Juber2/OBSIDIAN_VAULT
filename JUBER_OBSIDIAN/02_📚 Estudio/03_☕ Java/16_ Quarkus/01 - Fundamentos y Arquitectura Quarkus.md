---
tags:
  - Java
  - Quarkus
Creado: 2026-08-09
Relacionado:
  - Quarkus
---

# Fundamentos y Arquitectura Quarkus

## La idea central: mover trabajo de runtime a build-time

Spring Boot hace mucho de su trabajo (escaneo de classpath, proxies, reflection) **al arrancar la app**. Quarkus mueve ese trabajo al **momento de compilar** (build time), usando un modelo de "augmentation": procesa anotaciones, genera bytecode optimizado y arma el grafo de beans antes de que la app corra. El resultado: arranque en decenas de milisegundos en vez de segundos.

```
Spring Boot: código → arranque (escaneo + reflection + proxies) → app corriendo
Quarkus:     código → build-time processing (augmentation) → arranque casi instantáneo
```

## Extensions: el equivalente a starters de Spring

Quarkus se arma con "extensions" (análogo a `spring-boot-starter-*`), pero cada extension está optimizada para el modelo build-time.

```bash
mvn quarkus:add-extension -Dextensions="resteasy-reactive,hibernate-orm-panache,jdbc-postgresql"
```

## Crear un proyecto

```bash
mvn io.quarkus.platform:quarkus-maven-plugin:create \
  -DprojectGroupId=com.ejemplo \
  -DprojectArtifactId=mi-servicio \
  -Dextensions="resteasy-reactive-jackson"
```

O vía [code.quarkus.io](https://code.quarkus.io) (equivalente a Spring Initializr).

## Dev Mode: la diferencia más notable en el día a día

```bash
./mvnw quarkus:dev
```

Live reload real: cambiás código Java y el navegador/cliente ve el cambio sin reiniciar manualmente el proceso — más rápido que el DevTools de Spring Boot. Además expone una **Dev UI** (`/q/dev`) con inspección de beans, endpoints, config y extensions activas.

## Tres modos de ejecución

| Modo | Qué es | Cuándo se usa |
|---|---|---|
| JVM mode | corre normal sobre una JVM, pero con el bootstrap optimizado | default, balance entre arranque rápido y tooling JVM conocido |
| Native mode | compilado a binario nativo con GraalVM | arranque en milisegundos, mínimo footprint — serverless/K8s con autoscaling agresivo |
| Dev mode | hot reload para desarrollo | solo local |

## Errores comunes

- Asumir que todo el código Java corre igual que en Spring — algunas librerías que usan reflection dinámica (no declarada) fallan en compilación nativa si no se les da un "hint" explícito.
- No usar Dev Mode y perderse el live reload — es la mayor ganancia de productividad del día a día.
- Mezclar extensions incompatibles entre sí sin revisar la Dev UI, que muestra conflictos.

## Checklist
- [ ] Entiendo la diferencia entre procesamiento build-time (Quarkus) y runtime (Spring Boot clásico).
- [ ] Creé un proyecto Quarkus y usé `quarkus:dev`.
- [ ] Sé agregar extensions y verificar cuáles están activas en la Dev UI.
- [ ] Distingo JVM mode de Native mode y sé cuándo conviene cada uno.
