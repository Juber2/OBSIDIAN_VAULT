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

# 03 - Instalacion Docker Desktop vs Engine

## Que vas a aprender

En este capitulo vas a entender las distintas formas de instalar y correr Docker segun el sistema operativo: **Docker Engine** nativo en Linux, y **Docker Desktop** en Windows/Mac, incluyendo sus diferencias de licenciamiento y performance.

## Explicacion

En **Linux**, Docker Engine corre de forma nativa sobre el kernel del host, sin capas adicionales de virtualizacion, lo cual da la mejor performance posible. En **Windows y Mac**, como esos sistemas operativos no tienen el mismo kernel Linux que Docker necesita, **Docker Desktop** crea una VM liviana (usando WSL2 en Windows, o una VM ligera en Mac) donde realmente corre el daemon Linux; la experiencia se siente nativa gracias a la integracion, pero por debajo hay una capa de virtualizacion adicional que puede impactar en performance, especialmente en I/O de disco con bind mounts.

Es importante conocer tambien el cambio de licenciamiento de Docker Desktop: su uso gratuito tiene condiciones segun el tamaño de la empresa, lo cual ha llevado a muchos equipos a evaluar alternativas como **Podman** o Docker Engine directo sobre WSL2 en Windows.

## Modelo mental

Pensa Docker Engine en Linux como estar parado directamente sobre el terreno (el kernel), mientras que Docker Desktop en Windows/Mac es pararse sobre una plataforma elevada (la VM liviana) que a su vez esta sobre el terreno: funciona bien, pero hay una capa intermedia que puede introducir fricciones, especialmente notable en la velocidad de acceso a archivos montados desde el host.

## Ejemplo base

```bash
# Verificar que Docker esta corriendo y ver informacion del sistema
docker info

# En Windows, verificar que WSL2 esta siendo usado como backend
wsl --list --verbose
```

## Como se usa en un proyecto real

1. En Linux, se instala Docker Engine directo desde los repositorios oficiales, sin necesitar Docker Desktop.
2. En Windows, se configura Docker Desktop con backend WSL2 (no el legacy Hyper-V) para mejor performance e integracion con el sistema de archivos Linux.
3. Se evalua el licenciamiento de Docker Desktop segun el tamaño de la organizacion antes de asumir que es gratuito sin condiciones.
4. Se es consciente de que operaciones de I/O intensivas con bind mounts pueden ser mas lentas en Windows/Mac que en Linux nativo, y se planifica en consecuencia (por ejemplo, evitando montar `node_modules` completo desde el host en desarrollo).
5. Se documenta la version minima de Docker requerida por el equipo para evitar incompatibilidades de features entre distintas instalaciones.

## Criterio junior

Un junior debe poder instalar Docker en su sistema operativo siguiendo la documentacion oficial y verificar que funciona con `docker info`.

## Criterio mid

Un mid debe entender por que el performance de I/O con bind mounts puede diferir entre Linux nativo y Docker Desktop, y configurar WSL2 correctamente en Windows.

## Criterio senior

Un senior debe evaluar el modelo de licenciamiento de Docker Desktop para la organizacion, decidir si Docker Engine directo o alternativas como Podman son mas apropiadas segun el contexto, y estandarizar la configuracion de Docker para todo el equipo de desarrollo.

## Errores comunes

- No configurar WSL2 como backend en Windows, quedando con el backend legacy de Hyper-V, mas lento e integrado.
- Asumir que Docker Desktop es gratuito sin condiciones para cualquier organizacion, sin revisar los terminos de licencia actuales.
- Montar directorios pesados (como `node_modules`) desde el host en Windows/Mac sin anticipar el impacto de performance de I/O.
- No estandarizar la version de Docker entre el equipo, generando comportamientos distintos entre maquinas.

## Practica

- [ ] Instala Docker en tu sistema operativo (Engine en Linux, Desktop en Windows/Mac).
- [ ] Verifica la instalacion con `docker info` y `docker version`.
- [ ] Si estas en Windows, confirma que WSL2 es el backend activo.
- [ ] Investiga las condiciones actuales de licenciamiento de Docker Desktop para uso comercial.
- [ ] Compara la velocidad de un bind mount con muchos archivos chicos en tu entorno actual.

## Preguntas de repaso

- Por que Docker en Windows/Mac necesita una capa de virtualizacion adicional que Linux no necesita?
- Que ventaja de performance da WSL2 sobre el backend legacy de Hyper-V en Windows?
- Que implicancia tiene el licenciamiento de Docker Desktop para una empresa mediana o grande?
- Que tipo de operaciones se ven mas afectadas por la capa de virtualizacion en Docker Desktop?
