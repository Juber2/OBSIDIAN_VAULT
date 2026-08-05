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

# 05 - Comparativa Swarm vs Kubernetes vs Compose

## Que vas a aprender

En este capitulo vas a consolidar una comparativa practica entre las tres opciones vistas en este libro para correr containers en produccion: **Compose en un servidor**, **Swarm**, y **Kubernetes**, con criterios concretos para elegir entre ellas.

## Explicacion

Estas tres opciones no son estrictamente "niveles" donde una reemplaza a la anterior; representan puntos distintos en el espectro de simplicidad operativa versus capacidad y resiliencia. **Compose en un servidor** es la opcion mas simple: un unico punto de fallo, pero cero complejidad de orquestacion, perfecta para proyectos que no necesitan alta disponibilidad critica. **Swarm** agrega multi-nodo, replicas y rolling updates con una complejidad operativa moderada, usando herramientas ya familiares. **Kubernetes** ofrece el maximo de capacidades (auto-escalado avanzado, ecosistema enorme de operadores, adopcion de la industria) a cambio de la mayor complejidad operativa de las tres.

La eleccion correcta depende del tamaño del equipo, la experiencia operativa disponible, las necesidades reales de escala y disponibilidad, y el presupuesto (tanto de infraestructura como de tiempo de aprendizaje del equipo).

## Modelo mental

Pensa estas tres opciones como distintos medios de transporte para distintos trayectos: Compose en un servidor es caminar (simple, directo, suficiente para distancias cortas), Swarm es andar en auto propio (mas alcance y resiliencia, pero necesitas saber manejarlo), y Kubernetes es viajar en avion comercial (maxima capacidad y alcance, pero con toda la infraestructura de aeropuerto, personal especializado y regulaciones que eso implica).

## Ejemplo base

```
Tabla resumen de decision:

| Criterio                        | Compose (1 servidor) | Swarm            | Kubernetes         |
|----------------------------------|-----------------------|-------------------|---------------------|
| Complejidad operativa            | Muy baja              | Moderada          | Alta                |
| Tolerancia a fallo de servidor   | No                    | Si                | Si                  |
| Auto-escalado avanzado           | No                    | Basico            | Muy avanzado        |
| Ecosistema de herramientas       | El de Docker          | Limitado          | Enorme              |
| Curva de aprendizaje             | Baja (ya la sabes)    | Baja-Moderada     | Alta                |
| Adecuado para equipo pequeño     | Si                    | Si                | Solo con experiencia|
```

## Como se usa en un proyecto real

1. Se empieza por defecto con la opcion mas simple que resuelve el problema real actual, evitando adoptar complejidad especulativa "por si acaso".
2. Se usa la tabla comparativa como punto de partida de una decision documentada (ADR), no como respuesta automatica sin analisis del contexto especifico.
3. Se considera el costo de oportunidad: el tiempo que el equipo invertiria en aprender y operar Kubernetes podria invertirse en features de negocio si Compose o Swarm ya resuelven el problema real.
4. Se revisa la decision periodicamente, dado que las necesidades de un sistema cambian con el tiempo y lo que era correcto hace un año puede no serlo hoy.
5. Se documenta claramente para el equipo por que se eligio una opcion sobre las otras, evitando debates recurrentes sin nueva informacion.

## Criterio junior

Un junior debe poder nombrar la diferencia principal de complejidad y capacidad entre las tres opciones.

## Criterio mid

Un mid debe aplicar criterios concretos (tamaño de equipo, necesidades de escala, presupuesto de aprendizaje) para recomendar una opcion en un caso especifico.

## Criterio senior

Un senior debe liderar esta decision para la organizacion completa, documentarla formalmente, y revisarla periodicamente a medida que el sistema y el equipo evolucionan, evitando tanto la sobre-ingenieria prematura como quedarse atras de necesidades reales de escala.

## Errores comunes

- Elegir Kubernetes por defecto en cualquier proyecto nuevo sin evaluar si realmente se necesita su complejidad y capacidades.
- No revisar la decision con el tiempo, quedando atados a una opcion que ya no encaja con las necesidades actuales del sistema.
- Subestimar el costo de oportunidad de aprender y operar un orquestador complejo cuando una opcion mas simple resolveria el problema real.
- No documentar la decision tomada, generando debates repetidos sin nueva informacion cada vez que cambia alguien en el equipo.

## Practica

- [ ] Aplica la tabla comparativa a un proyecto real o hipotetico que conozcas, llegando a una recomendacion justificada.
- [ ] Documenta esa recomendacion como un ADR con los criterios considerados.
- [ ] Identifica el costo de oportunidad de aprender Kubernetes para tu equipo actual, frente al beneficio real esperado.
- [ ] Define que señales revisarias periodicamente para saber si la decision sigue siendo la correcta.
- [ ] Presenta (mentalmente o por escrito) esta decision como la explicarias a alguien que pregunta "por que no usamos Kubernetes directamente".

## Preguntas de repaso

- Que factor de decision, mas alla de lo puramente tecnico, es central para elegir entre estas tres opciones?
- Por que empezar con la opcion mas simple que resuelve el problema real, en vez de la mas capaz, suele ser la decision correcta?
- Que costo de oportunidad implica adoptar Kubernetes sin una necesidad real que lo justifique?
- Como revisarias periodicamente si la decision de orquestacion tomada sigue siendo la correcta para tu sistema?
