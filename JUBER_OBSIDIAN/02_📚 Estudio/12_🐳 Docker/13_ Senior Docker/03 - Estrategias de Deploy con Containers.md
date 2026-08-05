---
tags:
  - Docker
  - DevOps
  - Docker-Senior
Creado: 2026-07-28
Relacionado:
  - Senior Docker
  - DOCKER
---

# 03 - Estrategias de Deploy con Containers

## Que vas a aprender

En este capitulo vas a consolidar las estrategias de despliegue de bajo riesgo aplicadas a containers: **rolling update**, **blue-green**, y **canary**, con sus tradeoffs de complejidad y velocidad de rollback.

## Explicacion

**Rolling update** (ya vista en el capitulo de Orquestacion) reemplaza replicas de a poco, manteniendo el servicio disponible durante el despliegue, con rollback razonablemente rapido si algo falla. **Blue-green** mantiene dos entornos completos identicos ("azul" en produccion actual, "verde" con la nueva version), y el corte de trafico entre ambos es instantaneo (cambiar donde apunta el balanceador o el DNS), permitiendo un rollback practicamente inmediato con solo revertir el corte. **Canary** despliega la nueva version a un pequeño porcentaje del trafico real primero, observando metricas antes de aumentar gradualmente ese porcentaje, minimizando el impacto de un problema no detectado en testing.

Cada estrategia tiene un costo distinto: blue-green requiere el doble de infraestructura corriendo simultaneamente durante el despliegue; canary requiere mas sofisticacion de enrutamiento y observabilidad para decidir cuando avanzar o revertir automaticamente.

## Modelo mental

Pensa rolling update como remodelar un edificio ocupado piso por piso, sin cerrar el edificio completo. Blue-green es tener un edificio identico de respaldo completamente construido al lado, y simplemente redirigir a todos los visitantes hacia el nuevo edificio de una vez, pudiendo volver al anterior instantaneamente si algo sale mal. Canary es dejar que solo un pequeño grupo de visitantes prueben el edificio nuevo primero, observando si tienen problemas antes de invitar a todos.

## Ejemplo base

```bash
# Rolling update (Swarm, ya visto en capitulo de Orquestacion)
docker service update --image mi-registro/api:1.5.0 --update-parallelism 1 mi_servicio

# Blue-green: conceptual, cambiando a que grupo de containers apunta el proxy
# (verde ya corriendo con la nueva version, en paralelo con azul)
# cambio de configuracion del load balancer de "azul" a "verde"

# Canary: conceptual, con un proxy que soporta split de trafico por porcentaje
# 95% del trafico -> version estable, 5% -> version nueva, aumentando gradualmente
```

## Como se usa en un proyecto real

1. Se usa rolling update como default razonable para la mayoria de servicios, dado que balancea bien simplicidad y seguridad de despliegue.
2. Se evalua blue-green para servicios donde el rollback instantaneo es critico y el costo de doble infraestructura durante el despliegue es aceptable.
3. Se evalua canary para servicios de alto trafico donde detectar un problema con una fraccion pequeña de usuarios antes de un despliegue completo reduce significativamente el impacto de negocio.
4. Se combina cualquiera de estas estrategias con health checks reales (capitulo de Monitoreo) que determinen automaticamente si continuar o revertir un despliegue.
5. Se documenta que estrategia usa cada servicio y por que, como parte de las decisiones arquitectonicas del sistema.

## Criterio junior

Un junior debe poder explicar conceptualmente la diferencia entre rolling update, blue-green y canary.

## Criterio mid

Un mid debe implementar rolling updates con health checks reales que determinen automaticamente el exito o fallo de un despliegue.

## Criterio senior

Un senior debe decidir que estrategia de despliegue aplica a cada servicio segun su criticidad y trafico, e implementar blue-green o canary para los servicios que realmente lo justifiquen, considerando el costo de infraestructura y complejidad adicional.

## Errores comunes

- Usar la misma estrategia de despliegue para todos los servicios sin considerar su criticidad o volumen de trafico especifico.
- No conectar la estrategia de despliegue con health checks reales, perdiendo la capacidad de deteccion y reversion automatica ante un problema.
- Subestimar el costo de infraestructura de blue-green (doble capacidad durante el despliegue) al elegirlo sin evaluar el tradeoff.
- Implementar canary sin la observabilidad necesaria para decidir con datos cuando avanzar o revertir, perdiendo el beneficio real de la estrategia.

## Practica

- [ ] Implementa un rolling update con health checks reales para un servicio de practica.
- [ ] Diseña conceptualmente como implementarias blue-green para un servicio critico de tu sistema.
- [ ] Investiga que herramientas de tu stack de infraestructura soportarian despliegues canary con split de trafico.
- [ ] Evalua que estrategia de despliegue corresponde a cada servicio de un sistema real que conozcas.
- [ ] Documenta la estrategia de despliegue elegida para un servicio con su justificacion.

## Preguntas de repaso

- Que ventaja de rollback tiene blue-green sobre rolling update, y a que costo?
- Que problema especifico resuelve un despliegue canary que rolling update no resuelve?
- Como se conecta una estrategia de despliegue con los health checks vistos en el capitulo de Monitoreo?
- Que criterio usarias para decidir que estrategia de despliegue aplica a un servicio especifico?
