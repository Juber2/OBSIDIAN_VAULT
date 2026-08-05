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

# 05 - Troubleshooting Avanzado

## Que vas a aprender

En este capitulo vas a cerrar el libro con troubleshooting de nivel senior: diagnostico usando **namespaces y cgroups** directamente, problemas de networking dificiles de reproducir, y como pensar sistematicamente ante un problema que "solo pasa en produccion".

## Explicacion

Cuando el diagnostico estandar (capitulo de Monitoreo: logs, `docker inspect`, `docker exec`) no alcanza, entender los mecanismos de bajo nivel del kernel que Docker usa ayuda a diagnosticar problemas mas profundos. Los **namespaces** de Linux (PID, network, mount, entre otros) son lo que aisla un container; se puede inspeccionar el namespace de red de un container especifico con herramientas como `nsenter` para depurar problemas de conectividad complejos desde fuera del container. Los **cgroups** controlan y limitan recursos (ver tema anterior); revisar directamente los archivos de cgroups en `/sys/fs/cgroup/` puede revelar throttling de CPU o limites de memoria que no son obvios desde `docker stats`.

Para problemas que "solo pasan en produccion" (dificiles de reproducir en desarrollo), el enfoque sistematico es: aislar la variable diferente (carga real, datos reales, configuracion de red real), reproducir con la mayor fidelidad posible en un entorno de staging, y usar observabilidad exhaustiva (logs estructurados, metricas, traces) para capturar evidencia del problema la proxima vez que ocurra, en vez de depender solo de reproducirlo bajo demanda.

## Modelo mental

Pensa el troubleshooting avanzado como ser un mecanico que, cuando el diagnostico computarizado estandar del auto no encuentra el problema, sabe abrir el capot y revisar los componentes mecanicos reales (namespaces, cgroups) directamente, entendiendo como funcionan por debajo de la capa de abstraccion que normalmente uses.

## Ejemplo base

```bash
# Entrar al namespace de red de un container especifico desde el host, sin necesitar shell dentro del container
PID=$(docker inspect -f '{{.State.Pid}}' mi-container)
sudo nsenter -t $PID -n ip addr

# Revisar throttling de CPU directamente en cgroups (cgroup v2)
cat /sys/fs/cgroup/system.slice/docker-<container-id>.scope/cpu.stat
# nr_throttled y throttled_usec revelan si el container esta siendo limitado por su cuota de CPU
```

## Como se usa en un proyecto real

1. Se usa `nsenter` para diagnosticar problemas de red complejos en containers sin shell disponible (distroless) o cuando el problema es especificamente de la capa de red.
2. Se revisan metricas de cgroups directamente cuando se sospecha throttling de CPU que no es evidente desde herramientas de mas alto nivel.
3. Se invierte en observabilidad exhaustiva (logging estructurado, metricas, traces con request ID de correlacion, ver capitulo de Observabilidad en NestJS) precisamente para capturar evidencia de problemas dificiles de reproducir bajo demanda.
4. Se documenta cada problema complejo diagnosticado como un caso de estudio para el equipo, construyendo un repositorio de conocimiento de troubleshooting.
5. Se prioriza reproducir el problema con la mayor fidelidad posible en staging antes de experimentar directamente en produccion.

## Criterio junior

Un junior debe conocer que namespaces y cgroups son los mecanismos de bajo nivel detras del aislamiento y limites de un container, aunque no necesite operarlos directamente en el dia a dia.

## Criterio mid

Un mid debe poder usar `nsenter` para diagnosticar un problema de red complejo, y revisar metricas de cgroups cuando sospecha throttling de CPU.

## Criterio senior

Un senior debe liderar el diagnostico de problemas complejos que "solo pasan en produccion", invirtiendo en observabilidad exhaustiva del sistema para capturar evidencia, y construyendo un repositorio de conocimiento de troubleshooting para el equipo.

## Errores comunes

- Quedarse sin herramientas ante un problema que las herramientas de diagnostico estandar (logs, `docker exec`) no revelan, sin conocer alternativas de mas bajo nivel.
- No invertir en observabilidad exhaustiva, dependiendo de poder reproducir un problema bajo demanda para diagnosticarlo, lo cual no siempre es posible.
- Experimentar directamente en produccion para diagnosticar un problema complejo, en vez de intentar reproducirlo primero en un entorno de staging fiel.
- No documentar problemas complejos diagnosticados, perdiendo ese conocimiento para incidentes similares futuros.

## Practica

- [ ] Usa `nsenter` para inspeccionar el namespace de red de un container de practica desde el host.
- [ ] Revisa metricas de cgroups de un container para verificar si esta siendo throttled por CPU.
- [ ] Diseña una estrategia de observabilidad exhaustiva para capturar evidencia de un problema dificil de reproducir.
- [ ] Documenta un problema complejo (real o hipotetico) como caso de estudio de troubleshooting.
- [ ] Reflexiona sobre todo el libro: que temas revisarias primero ante un incidente real de produccion.

## Preguntas de repaso

- Que informacion revela `nsenter` que `docker exec` no siempre puede, especialmente en imagenes sin shell?
- Como usarias metricas de cgroups para diagnosticar throttling de CPU que no es obvio desde `docker stats`?
- Por que invertir en observabilidad exhaustiva es mas efectivo que depender de reproducir un problema bajo demanda?
- Que valor aporta documentar problemas complejos como casos de estudio para el equipo?

## Cierre del libro

Completar este capitulo cierra el recorrido completo de Docker: desde los fundamentos de que es un container, pasando por construccion de imagenes eficientes y seguras, redes, persistencia, orquestacion basica, CI/CD, y finalmente el criterio senior para tomar decisiones de arquitectura, costos y troubleshooting en produccion real. El proximo paso natural es aplicar este conocimiento en el pipeline de CI/CD y despliegue de tus propios proyectos, como [[NESTJS|el backend NestJS]] o StaffTrainingPlatform.
