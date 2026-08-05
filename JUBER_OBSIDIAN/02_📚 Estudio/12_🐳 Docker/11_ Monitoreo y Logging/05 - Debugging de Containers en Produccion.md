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

# 05 - Debugging de Containers en Produccion

## Que vas a aprender

En este capitulo vas a consolidar un flujo practico de **debugging de containers en produccion**: que comandos usar en orden ante un incidente, como diagnosticar un container que reinicia constantemente (crash loop), y como debuggear cuando no hay shell disponible (imagenes distroless, ver capitulo de Optimizacion).

## Explicacion

Ante un container problematico en produccion, el flujo tipico de diagnostico sigue un orden logico: primero `docker ps` (¿esta corriendo, reiniciando, o detenido?), despues `docker logs` (¿que dice la aplicacion sobre lo que le paso?), despues `docker inspect` (¿cual fue el codigo de salida, cuantas veces reinicio, que health status tiene?), y solo si es necesario `docker exec` para inspeccionar el estado interno de un container corriendo.

Un **crash loop** (el container arranca, falla, y Docker lo reinicia repetidamente segun la `restart_policy`) suele diagnosticarse revisando los logs del **ultimo intento fallido** especificamente (`docker logs --tail`), dado que el container puede haber reiniciado varias veces entre que el problema ocurrio y que alguien lo revisa. Cuando no hay shell disponible (distroless), el debugging depende mas fuertemente de logs exhaustivos y, si el orquestador lo soporta, de herramientas de debugging efimero (como `kubectl debug` en Kubernetes, que adjunta un container de debugging temporal sin modificar la imagen original).

## Modelo mental

Pensa el debugging de un container problematico como una investigacion con evidencia limitada: `docker ps` te dice si la escena esta activa o ya se disolvio, `docker logs` es el testimonio de lo que paso, `docker inspect` son los registros oficiales (hora exacta, cuantas veces paso antes), y `docker exec` es poder entrar fisicamente a la escena mientras todavia esta activa, algo que no siempre es posible.

## Ejemplo base

```bash
docker ps -a --filter name=mi-app          # estado actual, incluye si esta reiniciando
docker logs --tail 200 mi-app              # ultimos logs antes del ultimo reinicio
docker inspect --format '{{.RestartCount}} {{.State.ExitCode}}' mi-app
docker exec -it mi-app sh                  # solo si el container sigue corriendo y tiene shell

# Crash loop: revisar logs justo antes de que el container muriera
docker logs --since 5m mi-app
```

## Como se usa en un proyecto real

1. Se sigue un orden sistematico de diagnostico (`ps` -> `logs` -> `inspect` -> `exec`) en vez de saltar directo a intentar arreglar sin entender la causa.
2. Se revisan los logs con `--tail` o `--since` acotado al momento del ultimo fallo, dado que un crash loop puede haber generado logs de multiples intentos.
3. Se prepara de antemano una estrategia de debugging para imagenes sin shell (distroless), dado que en el momento del incidente ya es tarde para reconstruir la imagen con herramientas de debugging.
4. Se correlaciona el diagnostico local del container con logs centralizados y metricas (temas anteriores) para tener el panorama completo, no solo lo que ese unico container revela.
5. Se documenta cada incidente diagnosticado (causa raiz, sintomas, solucion) como aprendizaje para el equipo, evitando repetir el mismo proceso de diagnostico desde cero la proxima vez.

## Criterio junior

Un junior debe poder seguir el flujo basico de diagnostico (`ps`, `logs`, `inspect`) ante un container con problemas.

## Criterio mid

Un mid debe diagnosticar un crash loop revisando logs acotados al momento correcto, y usar `docker exec` efectivamente cuando esta disponible.

## Criterio senior

Un senior debe diseñar la estrategia de debuggability de toda la infraestructura (logging exhaustivo, herramientas de debugging efimero para imagenes sin shell) y liderar el proceso de post-mortem documentado despues de cada incidente relevante.

## Errores comunes

- Saltar directo a intentar "arreglar" un container problematico sin antes entender la causa raiz real del fallo.
- No revisar logs acotados al momento correcto en un crash loop, perdiendo la evidencia relevante entre reinicios sucesivos.
- No tener una estrategia de debugging preparada para imagenes distroless, quedando sin herramientas en el momento critico de un incidente.
- No documentar incidentes diagnosticados, repitiendo el mismo proceso de investigacion desde cero en incidentes similares futuros.

## Practica

- [ ] Simula un container que falla al arrancar y sigue el flujo completo de diagnostico (`ps`, `logs`, `inspect`).
- [ ] Provoca un crash loop y diagnostica la causa revisando logs acotados correctamente.
- [ ] Investiga como debuggearias un container distroless sin shell disponible.
- [ ] Correlaciona el diagnostico de un container especifico con logs centralizados y metricas de un stack de observabilidad.
- [ ] Documenta un incidente simulado con formato de post-mortem: causa raiz, sintomas, solucion aplicada.

## Preguntas de repaso

- En que orden logico revisarias `docker ps`, `docker logs`, `docker inspect` y `docker exec` ante un container problematico?
- Como diagnosticarias la causa de un crash loop cuando el container ya reinicio varias veces?
- Que estrategia de debugging usarias para un container basado en una imagen distroless sin shell?
- Que valor aporta documentar un incidente como post-mortem despues de resolverlo?
