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

# 05 - Ciclo de Vida de un Container

## Que vas a aprender

En este capitulo vas a entender los **estados por los que pasa un container** (created, running, paused, exited, dead), y como Docker maneja el proceso principal (PID 1) dentro de cada container.

## Explicacion

Un container pasa por estados bien definidos: **created** (existe pero no arranco), **running** (el proceso principal esta activo), **paused** (congelado con `docker pause`, sin consumir CPU pero manteniendo memoria), **exited** (el proceso principal termino, con un codigo de salida), y **dead** (un estado de error del que Docker no pudo limpiar correctamente).

Un concepto critico es que el container vive **mientras vive su proceso principal** (PID 1): si ese proceso termina, el container se detiene, sin importar si hay otros procesos hijos corriendo. Esto es distinto a una VM, que sigue "viva" independientemente de que procesos corran adentro. Ademas, el proceso PID 1 dentro de un container tiene responsabilidades especiales del kernel Linux (como reaping de procesos zombie) que muchas aplicaciones no manejan correctamente por si solas, por lo que se recomienda un init minimo (`--init` o `tini`) en ciertos casos.

## Modelo mental

Pensa el container como una burbuja que existe exactamente mientras su "corazon" (el proceso PID 1) sigue latiendo. Si ese corazon se detiene, no importa que otros organos (procesos hijos) sigan intentando funcionar, la burbuja entera se apaga.

## Ejemplo base

```bash
docker run -d --name test alpine sleep 30
docker inspect --format '{{.State.Status}}' test  # running

docker pause test
docker inspect --format '{{.State.Status}}' test  # paused

docker unpause test
# despues de 30s, el proceso sleep termina y el container pasa a exited

docker inspect --format '{{.State.ExitCode}}' test  # codigo de salida del proceso principal
```

## Como se usa en un proyecto real

1. Se diseña la aplicacion dentro del container para que el proceso principal maneje correctamente señales del sistema operativo (SIGTERM) para un shutdown graceful.
2. Se usa `--init` (o `tini` como ENTRYPOINT) cuando la aplicacion no maneja bien el reaping de procesos zombie por si sola, comun en apps Node.js o scripts que generan subprocesos.
3. Se interpreta el codigo de salida (`ExitCode`) del container para diagnosticar por que termino (0 = exito, distinto de 0 = algun tipo de error).
4. Se evita depender de `docker pause` para logica de negocio; es una herramienta de debugging/testing, no un mecanismo de control de flujo de produccion.
5. Se monitorea el estado de los containers en produccion para detectar reinicios inesperados (`docker inspect --format '{{.RestartCount}}'`).

## Criterio junior

Un junior debe poder identificar en que estado esta un container (`docker ps -a`) y entender que el container termina cuando termina su proceso principal.

## Criterio mid

Un mid debe interpretar codigos de salida para diagnosticar fallos, y saber cuando usar `--init` para aplicaciones que generan procesos hijos.

## Criterio senior

Un senior debe diseñar aplicaciones que manejen correctamente señales del sistema operativo para shutdown graceful, y establecer monitoreo del ciclo de vida de containers en produccion (reinicios, codigos de salida anomalos) como parte de la observabilidad del sistema.

## Errores comunes

- No manejar SIGTERM en la aplicacion, forzando a Docker a esperar el timeout completo y matar con SIGKILL, perdiendo la oportunidad de un cierre limpio.
- No usar `--init` en aplicaciones que generan procesos hijos, dejando procesos zombie sin ser "reaped" correctamente.
- Confundir `docker pause` con una herramienta de produccion, cuando es principalmente util para debugging o testing puntual.
- No monitorear reinicios inesperados de containers, perdiendo visibilidad de inestabilidad en produccion.

## Practica

- [ ] Corre un container simple y observa su transicion de estados con `docker inspect`.
- [ ] Provoca que un container termine con codigo de error distinto de 0 y diagnostica la causa.
- [ ] Investiga si tu aplicacion maneja correctamente SIGTERM para shutdown graceful.
- [ ] Prueba `--init` con una aplicacion que genera procesos hijos y observa la diferencia.
- [ ] Anota como monitorearias reinicios inesperados de containers en produccion.

## Preguntas de repaso

- Que determina que un container siga "vivo" o se detenga?
- Que diferencia hay entre los estados `paused` y `exited`?
- Por que algunas aplicaciones necesitan `--init` o `tini` como proceso principal?
- Como diagnosticarias por que un container termino con un codigo de salida distinto de 0?
