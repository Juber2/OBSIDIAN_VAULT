---
tags:
  - Docker
  - DevOps
  - Docker-Fundamentos
Creado: 2026-07-28
Relacionado:
  - Fundamentos de Contenedores
  - DOCKER
---

# 02 - Arquitectura de Docker Engine

## Que vas a aprender

En este capitulo vas a entender los componentes internos de **Docker Engine**: el daemon (`dockerd`), el cliente CLI, `containerd`, y `runc`, y como se comunican entre si para crear y correr un container.

## Explicacion

Docker no es un unico programa monolitico, sino una arquitectura en capas. El **cliente Docker** (`docker`) es la CLI que usas para dar comandos, que se comunican por API REST con el **daemon** (`dockerd`), el proceso que gestiona imagenes, redes y volumenes. El daemon delega la ejecucion real de containers a **containerd**, un runtime de alto nivel que a su vez usa **runc** (una implementacion de referencia de OCI, Open Container Initiative) para crear el container final usando namespaces y cgroups del kernel Linux.

Esta separacion en capas estandarizadas (OCI) es lo que permite que herramientas distintas (Docker, Kubernetes via containerd, Podman) puedan interoperar sobre el mismo tipo de imagenes y runtimes, sin estar atadas a una unica implementacion.

## Modelo mental

Pensa la arquitectura de Docker como una cadena de mando: vos le das una orden al cliente CLI (el "mensajero"), que se la comunica al daemon (el "gerente" que decide que hacer), que delega la tarea concreta a containerd (el "supervisor de obra"), que finalmente usa runc (el "obrero" que realmente construye el aislamiento usando las herramientas del kernel).

## Ejemplo base

```bash
# El cliente habla con el daemon via API REST (por defecto, un socket Unix local)
docker version   # muestra version del cliente Y del daemon (Server)

# Ver los containers realmente gestionados por containerd
sudo ctr containers list   # herramienta de bajo nivel de containerd, uso avanzado
```

## Como se usa en un proyecto real

1. Se entiende que problemas de Docker pueden originarse en distintas capas: el daemon no responde, containerd tiene un container zombie, o el propio kernel no soporta cierta feature.
2. Se aprovecha la compatibilidad OCI al elegir herramientas: una imagen construida con Docker corre igual en un cluster Kubernetes que usa containerd directamente.
3. Se diagnostica un problema de Docker revisando logs del daemon (`journalctl -u docker` en Linux) cuando el comportamiento no es el esperado.
4. Se entiende que reiniciar el daemon (`systemctl restart docker`) no necesariamente mata containers en ejecucion, gracias a la separacion con containerd.
5. Se documenta la version de Docker Engine usada en produccion, dado que la arquitectura interna evoluciona entre versiones mayores.

## Criterio junior

Un junior debe poder nombrar los componentes principales (cliente, daemon, containerd, runc) y entender que el cliente CLI no es quien ejecuta el container directamente.

## Criterio mid

Un mid debe poder diagnosticar en que capa de la arquitectura esta un problema (cliente, daemon, containerd, kernel) cuando algo no funciona como se espera.

## Criterio senior

Un senior debe entender las implicancias de la arquitectura OCI para decisiones de infraestructura (compatibilidad entre Docker, Kubernetes, Podman) y evaluar riesgos de seguridad relacionados con el daemon corriendo con privilegios elevados.

## Errores comunes

- Asumir que la CLI de Docker ejecuta containers directamente, sin entender la cadena real de responsabilidades.
- No saber donde buscar logs cuando el daemon falla, perdiendo tiempo de diagnostico.
- Ignorar que el daemon de Docker tradicionalmente corre con privilegios de root, lo cual tiene implicancias de seguridad (ver capitulo de Seguridad).
- Confundir "reiniciar Docker" con "matar todos los containers", cuando en versiones modernas containerd puede mantenerlos corriendo.

## Practica

- [ ] Corre `docker version` y identifica la version del cliente y del daemon (Server) por separado.
- [ ] Investiga como ver los logs del daemon de Docker en tu sistema operativo.
- [ ] Investiga que es OCI (Open Container Initiative) y por que su estandarizacion es relevante.
- [ ] Anota que pasa con los containers en ejecucion si reinicias el servicio del daemon.
- [ ] Diagrama de memoria la cadena cliente -> daemon -> containerd -> runc.

## Preguntas de repaso

- Que responsabilidad tiene el cliente Docker y en que se diferencia del daemon?
- Que es containerd y que rol cumple entre el daemon y runc?
- Por que la estandarizacion OCI permite interoperabilidad entre Docker y Kubernetes?
- Que implicancia de seguridad tiene que el daemon de Docker corra tradicionalmente como root?
