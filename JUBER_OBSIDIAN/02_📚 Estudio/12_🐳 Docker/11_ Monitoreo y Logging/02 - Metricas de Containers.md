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

# 02 - Metricas de Containers

## Que vas a aprender

En este capitulo vas a aprender a obtener y monitorear **metricas de recursos** de containers: `docker stats` para inspeccion rapida, y cAdvisor/herramientas equivalentes para monitoreo continuo integrado con un sistema de metricas.

## Explicacion

`docker stats` da una vista en tiempo real (pero puntual, no historica) de uso de CPU, memoria, I/O de red y disco por container, util para diagnostico rapido interactivo. Para monitoreo continuo real en produccion, se necesita algo que **recolecte estas metricas a lo largo del tiempo** y las haga consultables historicamente: **cAdvisor** (Container Advisor, de Google) es la herramienta clasica para esto, exponiendo metricas de containers en un formato que Prometheus puede recolectar (scraping) periodicamente.

Estas metricas son la base para decisiones concretas: detectar un container que consume memoria creciente (posible memory leak), dimensionar correctamente limites de recursos (ver siguiente consideracion practica), y alimentar reglas de auto-escalado basadas en uso real, no en suposiciones.

## Modelo mental

Pensa `docker stats` como mirar el velocimetro de un auto en el momento exacto que lo mirás: util pero solo te dice el instante actual. cAdvisor + Prometheus es como tener un registro de viaje completo (GPS con historial), que te permite ver patrones a lo largo del tiempo: cuando aceleraste, cuando frenaste, y detectar comportamientos anomalos que un vistazo puntual nunca revelaria.

## Ejemplo base

```bash
docker stats                           # vista en tiempo real de todos los containers
docker stats --no-stream mi-container  # una sola lectura puntual, util en scripts
```

```yaml
# cAdvisor como servicio que expone metricas para Prometheus
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports: ["8080:8080"]
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
```

## Como se usa en un proyecto real

1. Se usa `docker stats` para diagnostico rapido e interactivo cuando se sospecha un problema puntual de recursos.
2. Se despliega cAdvisor (u otra herramienta equivalente segun el stack de observabilidad del equipo) junto con Prometheus para monitoreo historico continuo en produccion.
3. Se definen alertas basadas en estas metricas (uso de memoria sostenido cerca del limite, CPU al 100% de forma prolongada) en vez de depender de revisiones manuales.
4. Se correlacionan picos de metricas con eventos de despliegue o trafico, para diagnosticar causas raiz de degradaciones de performance.
5. Se usan estas metricas historicas como input real para dimensionar limites de recursos (ver siguiente tema), en vez de asignar valores arbitrarios.

## Criterio junior

Un junior debe poder usar `docker stats` para revisar el uso de recursos de containers corriendo localmente.

## Criterio mid

Un mid debe desplegar cAdvisor (o equivalente) integrado con un sistema de metricas para monitoreo historico continuo de un entorno de produccion.

## Criterio senior

Un senior debe diseñar la estrategia de observabilidad de recursos de toda la infraestructura, definiendo alertas basadas en datos historicos reales y usando esas metricas para decisiones de capacity planning y auto-escalado.

## Errores comunes

- Depender solo de `docker stats` para monitoreo de produccion, perdiendo visibilidad historica y capacidad de detectar tendencias.
- No definir alertas basadas en metricas, descubriendo problemas de recursos solo cuando ya causaron un incidente visible.
- No correlacionar picos de metricas con eventos de despliegue, perdiendo la oportunidad de diagnosticar causas raiz rapidamente.
- Asignar limites de recursos sin datos historicos reales de consumo, adivinando valores en vez de basarse en evidencia.

## Practica

- [ ] Usa `docker stats` para revisar el consumo de recursos de containers en un entorno de practica.
- [ ] Despliega cAdvisor junto a un container de aplicacion y verifica que expone metricas.
- [ ] Investiga como conectarias cAdvisor con Prometheus para monitoreo historico.
- [ ] Define una alerta conceptual basada en uso de memoria sostenido cerca de un limite.
- [ ] Anota como usarias datos historicos de recursos para dimensionar limites de un servicio real.

## Preguntas de repaso

- Que limitacion tiene `docker stats` frente a un sistema de monitoreo historico continuo?
- Que rol cumple cAdvisor en un stack de observabilidad de containers?
- Por que definir alertas basadas en metricas es mejor que depender de revisiones manuales periodicas?
- Como usarias datos historicos de consumo de recursos para dimensionar limites de un servicio?
