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

# 02 - Inmutabilidad e Infraestructura como Codigo

## Que vas a aprender

En este capitulo vas a consolidar el principio de **inmutabilidad** en el contexto de containers: por que un container en produccion nunca deberia modificarse en caliente, y como esto se conecta con infraestructura como codigo.

## Explicacion

Un container **inmutable** significa que, una vez desplegado, nunca se le hacen cambios manuales directos (via `docker exec` para "arreglar algo rapido"): cualquier cambio necesario se hace modificando el Dockerfile o la configuracion, reconstruyendo una imagen nueva, y reemplazando el container viejo por uno nuevo. Esto garantiza que lo que corre en produccion siempre coincide exactamente con lo que esta versionado en el repositorio, eliminando el problema clasico de "servidores mascota" con configuracion acumulada a mano que nadie recuerda exactamente como se armo.

Este principio se extiende a **infraestructura como codigo**: la configuracion de despliegue (Compose files, manifiestos de Kubernetes, configuracion de red) tambien vive versionada en el repositorio, no como cambios manuales hechos directamente en el servidor de produccion, permitiendo revertir, auditar, y reproducir la infraestructura completa desde cero si fuera necesario.

## Modelo mental

Pensa un container inmutable como un producto de fabrica sellado: si algo esta mal, no lo abris y le soldas una pieza nueva a mano (modificacion en caliente); pedis un producto nuevo de la linea de produccion (nueva imagen) con el defecto corregido en el diseño, y reemplazas el producto completo. Cualquier "reparacion a mano" rompe la garantia de que todos los productos de la misma linea son identicos.

## Ejemplo base

```bash
# Anti-patron: "arreglar" algo dentro de un container de produccion corriendo
docker exec -it mi-api-prod sh
# vi config.json  <- cambio manual que se perdera y nadie documentara

# Patron correcto: el cambio va al codigo, se reconstruye, se reemplaza
git commit -am "fix: corrige valor de configuracion"
docker build -t mi-registro/api:sha-nuevo .
docker service update --image mi-registro/api:sha-nuevo mi-servicio
```

## Como se usa en un proyecto real

1. Se prohibe como practica de equipo modificar containers de produccion en caliente via `docker exec` para cambios permanentes.
2. Se versiona toda la configuracion de infraestructura (Compose, manifiestos) en el repositorio, tratandola con el mismo rigor que el codigo de la aplicacion.
3. Se diseñan los despliegues para ser reemplazables completos (rolling update, blue-green), nunca parches aplicados a instancias existentes.
4. Se usa `docker exec` solo para **inspeccion** de diagnostico (ver capitulo de Monitoreo), nunca para modificacion permanente de un container de produccion.
5. Se documenta y aplica este principio como parte de la cultura tecnica del equipo, conectando con el capitulo Senior del libro de NestJS sobre decisiones arquitectonicas documentadas.

## Criterio junior

Un junior debe entender por que modificar un container de produccion en caliente con `docker exec` es una mala practica, y que el cambio correcto siempre pasa por el codigo.

## Criterio mid

Un mid debe diseñar despliegues que sigan el principio de reemplazo completo (rolling update) en vez de parches en caliente sobre instancias existentes.

## Criterio senior

Un senior debe establecer la cultura de inmutabilidad e infraestructura como codigo en todo el equipo, auditando que ningun cambio permanente de produccion se haga fuera del flujo versionado de codigo y despliegue.

## Errores comunes

- Modificar un container de produccion en caliente para resolver un problema "rapido", perdiendo ese cambio la proxima vez que el container se recree.
- No versionar la configuracion de infraestructura, dependiendo de cambios manuales hechos directamente en servidores que nadie documenta.
- Confundir inspeccion de diagnostico (`docker exec` para mirar) con modificacion permanente (`docker exec` para cambiar algo y dejarlo asi).
- No tener un flujo claro de "el cambio va al codigo, se reconstruye, se reemplaza" bien establecido y respetado por todo el equipo.

## Practica

- [ ] Identifica si tu equipo tiene la practica de modificar containers de produccion en caliente, y documenta por que deberia evitarse.
- [ ] Verifica que toda tu configuracion de infraestructura (Compose, etc.) esta versionada en el repositorio.
- [ ] Diseña un flujo de "cambio va al codigo, se reconstruye, se reemplaza" para un caso real de tu proyecto.
- [ ] Practica usar `docker exec` solo para inspeccion, sin dejar cambios permanentes.
- [ ] Documenta el principio de inmutabilidad como una regla explicita para tu equipo.

## Preguntas de repaso

- Por que modificar un container de produccion en caliente con `docker exec` es problematico a largo plazo?
- Que significa tratar la configuracion de infraestructura como codigo versionado?
- Que diferencia hay entre usar `docker exec` para inspeccion y usarlo para modificacion permanente?
- Como conecta el principio de inmutabilidad con la capacidad de auditar y reproducir infraestructura desde cero?
