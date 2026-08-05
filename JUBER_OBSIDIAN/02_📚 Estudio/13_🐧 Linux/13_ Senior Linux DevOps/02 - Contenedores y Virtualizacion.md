---
tags:
  - Linux
  - DevOps
  - Linux-Senior
Creado: 2026-07-28
Relacionado:
  - Senior Linux DevOps
  - LINUX
---

# 02 - Contenedores y Virtualizacion

## Que vas a aprender

En este capitulo vas a consolidar la relacion entre todo lo aprendido en este libro y el libro de Docker: como namespaces y cgroups (vistos en el capitulo de Fundamentos de este libro) son la base tecnica real que hace posible la containerizacion, y cuando conviene virtualizacion completa versus containers.

## Explicacion

Este es el punto donde ambos libros se conectan explicitamente: los **namespaces** del kernel Linux (que aislan lo que un proceso puede ver: su propio arbol de procesos, su propia red, su propio filesystem) y los **cgroups** (que limitan cuantos recursos puede consumir un grupo de procesos: CPU, memoria) son exactamente los mecanismos que Docker usa internamente para crear el aislamiento de un container, ya mencionados de forma introductoria en el capitulo de Kernel de este libro.

Esto explica por que los containers son mucho mas livianos que una VM (comparten el kernel del host, solo aislan procesos con estos mecanismos) mientras que una **maquina virtual completa** (KVM, el hipervisor nativo de Linux) virtualiza hardware completo, incluyendo su propio kernel independiente. La eleccion entre containers y VMs completas depende del nivel de aislamiento realmente necesario: containers para la mayoria de cargas de aplicacion (rapidos, eficientes), VMs completas cuando se necesita aislamiento de kernel real (multi-tenant de alta seguridad, o correr un sistema operativo distinto al del host).

## Modelo mental

Pensa namespaces y cgroups como las paredes y el medidor de consumo de un departamento dentro de un edificio compartido (el kernel del host): separan lo que cada inquilino ve y limitan cuanto puede consumir, sin necesitar construir un edificio completo nuevo para cada inquilino. Una VM completa es literalmente construir un edificio nuevo independiente para cada inquilino, con sus propios cimientos (kernel) desde cero.

## Ejemplo base

```bash
# Ver los namespaces de un proceso especifico (incluyendo un container corriendo)
sudo ls -la /proc/$(pgrep -f mi-container)/ns/

# cgroups v2: verificar limites de un cgroup especifico
cat /sys/fs/cgroup/system.slice/docker-<id>.scope/memory.max

# KVM: verificar si el hardware soporta virtualizacion completa
egrep -c '(vmx|svm)' /proc/cpuinfo
```

## Como se usa en un proyecto real

1. Se elige containers (Docker, ver ese libro completo) para la gran mayoria de cargas de aplicacion, aprovechando su eficiencia y velocidad de arranque.
2. Se elige virtualizacion completa (KVM) solo cuando se necesita aislamiento de kernel real, multi-tenant de alta seguridad, o correr un sistema operativo distinto al del host.
3. Se investiga namespaces y cgroups directamente con herramientas de sistema cuando se necesita diagnosticar un problema de aislamiento o limite de recursos mas profundo de lo que Docker expone en su CLI.
4. Se conecta este conocimiento con todo el libro de Docker, entendiendo que cada feature de aislamiento de containers (redes, limites de recursos, usuarios) tiene una base tecnica directa en mecanismos del kernel Linux ya estudiados aca.
5. Se documenta esta relacion tecnica para el equipo, ayudando a que quienes usan Docker diariamente entiendan que hay detras de la abstraccion que usan.

## Criterio junior

Un junior debe entender conceptualmente que namespaces y cgroups son los mecanismos del kernel que hacen posible el aislamiento de containers.

## Criterio mid

Un mid debe inspeccionar namespaces y cgroups directamente con herramientas de sistema para diagnosticar problemas de aislamiento o recursos mas profundos que lo que expone el CLI de Docker.

## Criterio senior

Un senior debe decidir con criterio tecnico solido cuando usar containers versus virtualizacion completa, entendiendo profundamente el tradeoff real de aislamiento versus eficiencia en cada nivel.

## Errores comunes

- No entender la base tecnica real detras de Docker, tratandolo como "magia" en vez de una aplicacion practica de mecanismos del kernel ya estudiados.
- Elegir virtualizacion completa por defecto sin necesidad real de ese nivel de aislamiento, perdiendo la eficiencia que dan los containers.
- No saber inspeccionar namespaces/cgroups directamente cuando el CLI de Docker no da suficiente informacion para un diagnostico profundo.
- Confundir el aislamiento de containers (comparten kernel) con el aislamiento de VMs completas (kernel independiente), subestimando riesgos de seguridad en entornos multi-tenant sensibles.

## Practica

- [ ] Inspecciona los namespaces de un proceso o container corriendo con `/proc/PID/ns/`.
- [ ] Revisa los limites de cgroups de un container Docker corriendo, comparando con lo configurado via `docker run --memory`.
- [ ] Verifica si tu hardware soporta virtualizacion completa (KVM) con `/proc/cpuinfo`.
- [ ] Explica con tus palabras por que un container arranca mas rapido que una VM completa.
- [ ] Documenta la relacion entre namespaces/cgroups y las features de Docker que ya conoces del libro correspondiente.

## Preguntas de repaso

- Que mecanismo del kernel aisla lo que un proceso puede ver (su propia red, su propio arbol de procesos)?
- Que mecanismo del kernel limita cuantos recursos puede consumir un grupo de procesos?
- Por que una VM completa es mas pesada que un container, en terminos de lo que cada uno virtualiza?
- Cuando elegirias virtualizacion completa sobre containers para una carga especifica?
