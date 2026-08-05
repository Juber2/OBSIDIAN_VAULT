---
tags:
  - Docker
  - DevOps
  - Docker-Monitoreo
Creado: 2026-07-28
Relacionado:
  - Monitoreo y Logging
  - DOCKER
---

# 04 - Health Checks

## Que vas a aprender

En este capitulo vas a profundizar en la instruccion **`HEALTHCHECK`** del Dockerfile: como definirla correctamente, y como Docker/Compose/Swarm usan su resultado para tomar decisiones automaticas.

## Explicacion

`HEALTHCHECK` define un comando que Docker ejecuta periodicamente **dentro** del container para determinar si la aplicacion esta realmente sana, mas alla de si el proceso simplemente sigue corriendo. Sin un health check, Docker solo sabe si el proceso principal esta vivo o muerto (ver capitulo de Ciclo de Vida), pero no si esta realmente respondiendo correctamente (por ejemplo, un servidor web podria estar "corriendo" pero colgado, sin responder a ningun request).

El resultado del health check (`healthy`, `unhealthy`, `starting`) es lo que consumen otros mecanismos ya vistos: `depends_on: condition: service_healthy` en Compose (capitulo de Redes), y en Swarm/Kubernetes, un container `unhealthy` puede disparar automaticamente su reemplazo por una instancia nueva.

## Modelo mental

Pensa `HEALTHCHECK` como un chequeo medico periodico, no solo verificar que la persona respira (proceso vivo), sino tomar signos vitales reales (responde correctamente a un estimulo especifico) para decidir si esta en condiciones de seguir trabajando o necesita ser reemplazada.

## Ejemplo base

```dockerfile
FROM nginx:alpine
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD curl -f http://localhost/health || exit 1
```

```bash
docker inspect --format '{{.State.Health.Status}}' mi-container
# healthy, unhealthy, o starting
```

## Como se usa en un proyecto real

1. Se define un endpoint de salud real en la aplicacion (ver capitulo de Observabilidad en NestJS) que el `HEALTHCHECK` consulta, verificando dependencias criticas (base de datos, cache), no solo respondiendo un `200 OK` fijo.
2. Se configura `start-period` para dar tiempo suficiente a que la aplicacion complete su inicializacion antes de que los primeros chequeos fallidos cuenten como reales.
3. Se ajusta `interval` y `retries` segun la criticidad del servicio: chequeos mas frecuentes para servicios criticos, con cuidado de no generar carga innecesaria.
4. Se aprovecha el estado `unhealthy` en el orquestador (Swarm, Kubernetes) para que reemplace automaticamente instancias que dejaron de responder correctamente.
5. Se evita un health check que solo verifica que el proceso responde algo, sin verificar realmente las dependencias criticas de las que depende la funcionalidad core.

## Criterio junior

Un junior debe poder definir un `HEALTHCHECK` basico en un Dockerfile y verificar su estado con `docker inspect`.

## Criterio mid

Un mid debe diseñar un endpoint de salud que verifique dependencias reales, y ajustar los parametros de `HEALTHCHECK` (interval, timeout, retries, start-period) segun las caracteristicas de la aplicacion.

## Criterio senior

Un senior debe definir el estandar de health checks de toda la infraestructura, asegurando que cada servicio critico tenga un chequeo real y utilizado activamente por el orquestador para decisiones automaticas de reemplazo.

## Errores comunes

- Definir un `HEALTHCHECK` trivial que siempre responde exitoso, sin valor real como señal de salud.
- No configurar `start-period`, generando falsos negativos de salud durante la inicializacion normal de la aplicacion.
- Verificar solo que el proceso responde, sin chequear dependencias criticas reales (base de datos, cache) de las que depende la funcionalidad.
- No aprovechar el estado `unhealthy` en el orquestador, perdiendo la oportunidad de reemplazo automatico de instancias problematicas.

## Practica

- [ ] Define un `HEALTHCHECK` en un Dockerfile que verifique un endpoint de salud real de la aplicacion.
- [ ] Ajusta `start-period` para dar tiempo suficiente a la inicializacion de esa aplicacion.
- [ ] Verifica el estado de salud con `docker inspect` en distintos momentos del ciclo de vida del container.
- [ ] Simula un fallo de dependencia (por ejemplo, apagar la base de datos) y observa que el health check pasa a `unhealthy`.
- [ ] Investiga como tu orquestador (Compose, Swarm, o el que uses) reacciona ante un container `unhealthy`.

## Preguntas de repaso

- Que diferencia hay entre un proceso "vivo" y un container "healthy" segun Docker?
- Para que sirve el parametro `start-period` en `HEALTHCHECK`?
- Que deberia verificar un buen endpoint de salud, mas alla de responder un `200 OK` fijo?
- Como aprovecha un orquestador el estado `unhealthy` de un container?
