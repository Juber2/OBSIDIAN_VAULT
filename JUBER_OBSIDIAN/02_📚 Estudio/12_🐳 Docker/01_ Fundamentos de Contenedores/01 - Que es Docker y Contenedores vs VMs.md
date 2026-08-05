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

# 01 - Que es Docker y Contenedores vs VMs

## Que vas a aprender

En este capitulo vas a entender **que es Docker**, que problema resuelve, y la diferencia fundamental entre un **contenedor** y una **maquina virtual**, que es la base de todo lo que sigue en este libro.

## Explicacion

Docker es una plataforma que empaqueta una aplicacion junto con todas sus dependencias (librerias, runtime, configuracion) en una unidad portable llamada **imagen**, que corre de forma aislada como **container**. El problema clasico que resuelve es "funciona en mi maquina": si la imagen incluye todo lo necesario, el comportamiento es identico en cualquier entorno que tenga Docker instalado.

La diferencia clave con una **maquina virtual** es el nivel de aislamiento: una VM virtualiza hardware completo y corre un sistema operativo entero por encima de un hipervisor, mientras que un container comparte el kernel del sistema operativo anfitrion, aislando solo procesos, red y filesystem mediante mecanismos del kernel Linux (namespaces y cgroups). Esto hace que los containers sean mucho mas livianos y rapidos de iniciar que una VM.

## Modelo mental

Pensa una VM como un edificio completo nuevo, con su propia estructura, instalaciones y cimientos (kernel propio). Pensa un container como un departamento dentro de un edificio ya existente: comparte la estructura del edificio (el kernel del host), pero tiene su propia puerta, sus propias paredes y su propio espacio aislado del resto de los departamentos.

## Ejemplo base

```bash
docker run hello-world
# Descarga la imagen (si no existe local), crea un container, lo ejecuta, y termina

docker run -d -p 8080:80 nginx
# Corre nginx en background, mapeando el puerto 80 del container al 8080 del host
```

## Como se usa en un proyecto real

1. Se usa Docker para garantizar que el entorno de desarrollo, testing y produccion sean identicos.
2. Se eligen containers sobre VMs cuando se necesita velocidad de arranque, densidad (muchos servicios por servidor) y portabilidad simple entre entornos.
3. Se reserva el uso de VMs para casos que necesitan aislamiento de kernel completo (multi-tenant con requisitos de seguridad extremos) o sistemas operativos distintos al del host.
4. Se documenta la version minima de Docker requerida por el proyecto, dado que las features disponibles varian entre versiones.
5. Se entiende que Docker en Windows/Mac corre en realidad sobre una VM liviana (Docker Desktop), mientras que en Linux corre nativo sobre el kernel del host.

## Criterio junior

Un junior debe poder explicar con sus palabras la diferencia entre un container y una VM, y ejecutar su primer container con `docker run`.

## Criterio mid

Un mid debe poder justificar cuando usar containers y cuando una VM sigue siendo la opcion correcta, considerando aislamiento, portabilidad y densidad de recursos.

## Criterio senior

Un senior debe decidir la estrategia de contenerizacion de un sistema completo, entendiendo los limites reales de aislamiento que dan los containers (comparten kernel) frente a los riesgos de seguridad que eso implica en entornos multi-tenant.

## Errores comunes

- Asumir que un container da el mismo nivel de aislamiento de seguridad que una VM, cuando en realidad comparten el kernel del host.
- Contenerizar cargas de trabajo que realmente necesitan un kernel o sistema operativo distinto al del host sin evaluar esa limitacion.
- No entender que en Windows/Mac, Docker corre sobre una VM liviana, lo cual explica diferencias de performance frente a Linux nativo.
- Confundir "portable" con "identico en cualquier hardware": la arquitectura de CPU (x86 vs ARM) todavia importa (ver capitulo de multi-arch builds).

## Practica

- [ ] Instala Docker y corre `docker run hello-world`.
- [ ] Corre un container de nginx mapeando un puerto y accede desde el navegador.
- [ ] Investiga y anota tres diferencias tecnicas concretas entre containers y VMs.
- [ ] Identifica un caso de tu experiencia donde una VM seria mas apropiada que un container.
- [ ] Verifica si tu Docker corre nativo sobre Linux o sobre una VM liviana (Docker Desktop).

## Preguntas de repaso

- Que comparte un container con el sistema host que una VM no comparte?
- Por que los containers arrancan mucho mas rapido que una VM?
- En que escenario elegirias una VM sobre un container?
- Que problema clasico de desarrollo resuelve Docker al empaquetar dependencias junto a la aplicacion?
