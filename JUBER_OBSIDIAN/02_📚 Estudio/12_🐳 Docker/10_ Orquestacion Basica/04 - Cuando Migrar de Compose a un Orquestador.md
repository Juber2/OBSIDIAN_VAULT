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

# 04 - Cuando Migrar de Compose a un Orquestador

## Que vas a aprender

En este capitulo vas a desarrollar el criterio para decidir **cuando** un proyecto que corre bien con Docker Compose en un unico servidor necesita realmente migrar a un orquestador real (Swarm o Kubernetes), y cuando esa migracion seria prematura.

## Explicacion

Docker Compose en un unico servidor (potencialmente con `restart: always` y un buen monitoreo) es una solucion completamente valida y usada en produccion para muchos proyectos reales, especialmente los que no necesitan alta disponibilidad de nivel critico. Las señales concretas que justifican migrar a un orquestador son: la necesidad real de **tolerancia a fallos de servidor completo** (si el unico servidor se cae, todo el sistema se cae), necesidad de **escalar horizontalmente** mas alla de lo que un servidor puede manejar, o requisitos de **disponibilidad** (SLA) que un unico punto de fallo no puede cumplir.

Migrar prematuramente, sin estas necesidades reales, paga el costo de complejidad operativa (aprender el orquestador, mantenerlo, diagnosticar problemas distribuidos) sin el beneficio correspondiente. Este es el mismo tipo de analisis visto para arquitectura en el capitulo Senior de NestJS: la pregunta no es "es esto mejor practica en abstracto", sino "este problema especifico, en este momento, justifica esta complejidad".

## Modelo mental

Pensa Docker Compose en un servidor unico como andar en bicicleta: simple, eficiente, y perfectamente adecuado para trayectos cortos y medianos. Un orquestador es como pasar a un vehiculo con chofer profesional dedicado: te da mucha mas capacidad y resiliencia, pero necesitas justificar ese costo adicional con un trayecto (una necesidad real) que efectivamente lo requiera.

## Ejemplo base

```
Marco de decision: migrar de Compose a un orquestador si...

[ ] Una caida del unico servidor actual tiene un costo de negocio inaceptable.
[ ] La carga actual o proyectada supera lo que un servidor razonable puede manejar.
[ ] Existe un SLA formal de disponibilidad que un punto unico de fallo no puede cumplir.
[ ] El equipo tiene (o puede adquirir con tiempo razonable) la experiencia operativa necesaria.

Si ninguna de estas aplica hoy: Compose en un servidor bien monitoreado sigue siendo
una decision solida y defendible, no una limitacion a "arreglar" por moda.
```

## Como se usa en un proyecto real

1. Se evalua honestamente el impacto real de negocio de una caida antes de asumir que se necesita alta disponibilidad de nivel orquestador.
2. Se mide la carga real del sistema frente a la capacidad de un servidor bien dimensionado antes de asumir que se necesita escalado horizontal automatico.
3. Se documenta esta decision como un ADR (ver capitulo Senior de NestJS), revisable cuando el contexto cambie.
4. Se prepara el sistema para una eventual migracion (aplicaciones stateless, ver capitulo de Escalabilidad de NestJS) incluso si hoy corre en Compose, para que el camino de escalado futuro no requiera una reescritura completa.
5. Se revisa esta decision periodicamente a medida que el sistema y el negocio crecen, sin aferrarse indefinidamente a la decision original si el contexto cambio.

## Criterio junior

Un junior debe entender que Docker Compose en un servidor unico es una opcion de produccion valida para muchos proyectos, no solo una herramienta de desarrollo.

## Criterio mid

Un mid debe evaluar las señales concretas (tolerancia a fallos, escala, SLA) que justificarian migrar a un orquestador para un proyecto especifico.

## Criterio senior

Un senior debe liderar esta decision arquitectonica con el mismo rigor que cualquier otra decision de arquitectura significativa, documentandola, revisandola periodicamente, y preparando el sistema para una migracion futura sin necesitar reescribirlo desde cero cuando llegue el momento.

## Errores comunes

- Migrar a un orquestador por moda o presion externa, sin una necesidad real de negocio que lo justifique hoy.
- Quedarse en Compose de un solo servidor cuando las señales de necesidad real (caidas costosas, carga creciente) ya son evidentes, por resistencia al cambio.
- No preparar la aplicacion para ser stateless desde el principio, complicando innecesariamente una eventual migracion futura.
- No revisar esta decision periodicamente, quedando atado a una eleccion que ya no encaja con la realidad actual del negocio.

## Practica

- [ ] Aplica el marco de decision a un proyecto real que corre en Compose y determina si justifica migrar a un orquestador.
- [ ] Documenta esa decision (migrar o no) como un ADR con las señales consideradas.
- [ ] Verifica si la aplicacion de ese proyecto es realmente stateless, facilitando una futura migracion si fuera necesaria.
- [ ] Identifica que metricas revisarias periodicamente para saber si la decision sigue siendo valida con el tiempo.
- [ ] Anota un ejemplo de proyecto (real o hipotetico) donde Compose en un servidor unico sigue siendo la decision correcta hoy.

## Preguntas de repaso

- Por que Docker Compose en un servidor unico sigue siendo una opcion valida de produccion para muchos proyectos?
- Que señales concretas justificarian migrar de Compose a un orquestador real?
- Que riesgo tiene migrar a un orquestador sin una necesidad real que lo justifique?
- Como prepararias una aplicacion en Compose para que una eventual migracion futura sea mas simple?
